# OOP on the Z80: Structures, vtables, and Types Without Overhead

*Notes on an abandoned project: a Lisp meta-assembler for the Zilog Z80 that knew about types*

I abandoned this project. But along the way, a few ideas emerged that still seem interesting to me for anyone who loves the Z80: how to give an 8-bit machine structures, inheritance, and virtual methods without losing a single extra clock cycle and without drowning in magic numbers. Below is what managed to work, and an honest note of what didn't.

![Z80 OOP](../images/oop_z80.webp)

## 1. How It All Started: Drivers for Aleste

The original task was down-to-earth. For the Aleste LX platform and its BIOS, I was writing a specification for drivers (video, audio, file systems) for a Z80 at roughly 4 MHz with banked memory. The architecture's motto: **speed through predictability**.

The platform's main problem is that there are many video modes and they contradict each other. Ordinary code turns into a forest of `if/else` checks, and each check costs dozens of clock cycles. The solution is this:

- a polymorphic driver has a fixed slot in memory: a vtable (the slot is aligned to 64 bytes for round addresses; the minimum is 15 bytes: five mandatory methods) made of `JP` instructions of three bytes each;
- the application always calls the labels of this slot, and when the mode changes, the implementation is copied into the slot once (`LDIR`), after which the call remains direct, with no checks; a simple driver copies just 15 bytes, a complex one more;
- the overhead of a call through the table: one JP (10 cycles) and a RET (another 10), that is, about 20 cycles (the figure in the specification, 17, still needs to be confirmed);
- singletons (MMU, interrupt controller, system timer) are called directly at their own labels and copy nothing;
- memory banks are switched lazily, only when truly needed.

On the forum I was asked whether this was all about a Z80 at 4 MHz. Yes. And I called this "polymorphism" with a caveat: it's not OOP for OOP's sake, but **a rejection of checks**. A side benefit is that a program written for one video mode automatically works in any other: the code thinks about logic, not about hardware.

### What the BIOS Specification Says

All of this is described in the draft Aleste BIOS driver specification. A few of its rules will be needed later:

- **Five mandatory methods** for any driver: `detect()`, `init()`, `deinit()`, `get_info()`, and `command()`. Power, mode switching, and the like go into `command()` by command code, so that the vtable doesn't grow.
- **Three states**: ABSENT, PRESENT, READY. Each allows its own set of methods, and device-specific methods are available only in READY.
- **A vtable of 15 bytes or more**: five mandatory methods of three bytes each and up to 16 optional ones. The 64-byte slot is needed only for alignment, so that addresses come out round. Unused entries point to a common `_undefined_method` handler.
- **Memory context**: a driver has an 8-byte `MemoryContext` structure with an XOR hash of the bank configuration. Before a call, the hashes are compared: if they match, nothing needs to be switched (5–8 cycles); if they don't, the windows are reconfigured (up to ~50 cycles).
- **Errors**: success means CF=0, failure means CF=1 and an error code in register A.
- **The KISS principle**: better not to declare a function, variable, or constant than to declare one without an immediate purpose.

A minimal driver consists of only the five mandatory methods: all control goes through `init()` and the universal `command()`, into which the command code is passed in the accumulator. For hot operations (draw a sprite, output a character), this path is too slow, so the driver adds optional methods with a fast entry point directly inside. A simple driver (switch mode, output text) needs only 15 bytes of table; the more complex the driver, the more bytes are copied on activation.

## 2. The Hell of Synchronization

The first implementation was a mix of C and assembly. When the main modules were written, the real cost of this approach became apparent: any change to a data structure had to be manually dragged along the chain

`.h` → `.c` → `.inc` → `.asm`

Field offsets "drifted," registers and method arguments stopped matching each other. The worst part was not the errors themselves, but the fact that they had to be caught on working hardware.

The first thought was simple: generate files from annotations in the assembly code (that's how the XIFF specification came about). But the annotations themselves mutated into S-expressions as work progressed, and then the question arose: **why annotations, if you can write everything in Lisp?** Macros and a DSL cover exactly the task for which I was building generators.

## 3. A Z80 Assembler as a Lisp Macro

The assembler was first written in SOOT (my Lisp dialect), then ported to a base from the Common Lisp library, and eventually rewritten from scratch. The previous version was good but convoluted: several levels of macros generated a scattering of methods, each for a single operation signature.

Assembly code is ordinary S-expressions, so the entire language is available inside it:

```lisp
(defmacro subroutine (name &rest code)
  `(list (lab ',name) ,@code))

(zasm
  $start
    (jp 'label3)
    (db 22)
    (db (list 22 33 44))
    (dw 64)
    (ascii "Hello World\n")
    (align 16)
  $label1
    (jmp 'label1)               ; will choose jp or jr itself
    (ld @c (+ 1 3))             ; computation at compile time
  (subroutine label3
    (ld @a [+ ix 5])            ; DD 7E 05
    (ld [- iy 10] #xAA)))       ; FD 36 F6 AA
```

What the assembler did on its own:

- chose between `jp` and `jr` and calculated jump offsets;
- evaluated expressions at compile time;
- supported indexed addressing `(ix+d)` and `(iy+d)`;
- assembled modules from several segments, produced a listing and Intel HEX along with a symbol table.

Diagnostics were a separate priority. Macro generation without a precise error location turns into torture, so the message looked like this:

```
─── ERROR ──────────────────────────────────
  at common/xiff/soot/asm-test.sot:48
        (jmp 'blo-undefined)
        ^
Error: Can't find label blo-undefined
```

## 4. A Type System That Knows About Offsets

The main idea: the assembler should know as much about types as a high-level language compiler does. A structure is described once:

```lisp
(deftype vec3 (structure)
  ((x int16)
   (y int16)
   (z int16))
  (:methods
    (new (int16 int16) none)
    (len () int16)))
```

After that, the environment knows the sizes, the hierarchy, and the field offsets with alignment taken into account, and all of this can be used directly in code:

```lisp
(ld @hl (size-of vec3))
(ld @hl (offset-of vec3 y))
(ld [+ iy (offset-of vec3 y)] 10)   ; → FD 36 02 0A
```

Instead of magic numbers, field names remain, and the output is pure constants. If you replace `int16` with `int32`, all offsets in all files are recalculated at the next build. There is no runtime overhead at all: the type exists only at compile time.

A more familiar notation uses the `rlet` form, which binds an object to a register:

```lisp
(defmethod clear ((self vec3))
  (rlet ((v :reg ix :source self))
    (zasm
      (.ld (-> v x) 0)
      (.ld (-> v y) 0)
      (.ld (-> v z) 0))))
```

Result:

```
DD 36 00 00 | LD (IX+0), $00
DD 36 02 00 | LD (IX+2), $00
DD 36 04 00 | LD (IX+4), $00
```

Three practical consequences:

- **you don't need to remember offsets**: you write `(-> v y)`, and the compiler substitutes the numbers;
- **errors are caught at build time**: a nonexistent field breaks the build immediately, not a running program;
- **code isn't tied to a register**: replacing `ix` with `iy` takes one line of `rlet`.

### Introspection

The environment itself can also be explored. The `->` operator is implemented as a special form on the C++ side and lets you "walk" through the type registry directly from the REPL:

```lisp
soot> (-> *type-system* pointer-size)
=> 2
soot> (-> *type-system* vec3 x offset)
=> 0
soot> (-> *type-system* int8 size)
=> 1
```

Under the hood, this is a chain of virtual `make_step_alias` calls along the class hierarchy (`Aliasable` → `StructureType` → `BasicType`), and the data depends on the chosen architecture (Z80 or default values). For this, I had to carefully separate "field type" and "type specification" so that navigation wouldn't confuse the levels.

## 5. Virtual Methods on the Z80

Now for the most interesting part: what an object model looks like on a processor that has neither a frame stack nor an indirect call.

Let me say upfront: in this scheme there are two different polymorphism mechanisms, and they are easy to confuse.

**The driver one** works through a slot. The `CMD_SWITCH_DRIVER` command copies the vtable of the selected implementation into a fixed location, and the application always calls the slot's labels (for example, `__video_draw_pixel`). Switching costs a copy, and the calls themselves are free.

**The object one** works through the type. The first two bytes of an object hold a pointer to its type's vtable, and the method is taken by index. Nothing needs to be switched, but each call pays for looking up the address.

The layout is the same for both: `JP` triples of three bytes each. Therefore the `vtable` metacommand could build both driver slots and type tables. That's what the project was started for: to move the drivers onto the new system.

### The Table as a Sequence of Jumps

Here a vtable is an array of `JP addr` instructions (opcode `C3`), three bytes per method. The method index is known from `deftype`, so the `vtable` metacommand places the jumps itself:

```lisp
$vec3::new
      (ret)
$vec3::len
      (ret)
      (align 16)
      (vtable position vec3)
```

It looks at the type, finds the implemented methods, arranges them in the correct order, and fills the "holes" for methods that aren't defined. Add a method to `deftype`, and the table rebuilds itself.

### The Type Descriptor and Inheritance

The `.type-descriptor` macro generates a compact data structure for each type:

- a reference to the parent descriptor;
- the type name as a string (for debugging);
- flags and a type identifier;
- its own vtable.

```lisp
(defproc test-type-descriptor ()
  (declare (once) (asm-func none))
  (.type-descriptor object)
  (.type-descriptor structure)
  (.type-descriptor vec3))
```

And in the listing:

```
[type::def::vec3]
  DW <type::def::structure>     ; pointer to parent
[vec3::vtable::new]
  JP vec3::new                  ; overridden method
  JP object::delete             ; inherited method
```

Inheritance comes out "free": if a method isn't overridden in a descendant, a `JP` to the parent's implementation is simply copied into its table. Having names and references to parents at runtime makes it possible to do type checking directly on the hardware.

### The Calling Convention

For methods I adopted these rules:

- arguments are passed sequentially through `AF/HL`, `DE`, `BC`;
- `IX` points to the object;
- `IY` points to the vtable (for dynamic dispatch);
- the result is returned in `AF/HL`.

A procedure declares the argument types along with the registers, and the assembler checks them at the call:

```lisp
(defproc test-copy-string ((dest string :reg de)
                           (src  string :reg hl))
  (declare (once) (asm-func none))
  (.label 'loop)
  (.ld a (@ hl))
  (.ld (@ de) a)
  (.or a)
  (.ret 'z)
  (.inc hl)
  (.inc de)
  (.jr 'loop))
```

### Three Ways to Call

The `dcall` primitive checks the types of the arguments and the result and supports three variants:

```lisp
(dcall vec3::new int16 int16 _type_)
```

1. **Direct call**, if the function is known statically: an ordinary `CALL`, no overhead.
2. **Through the type's vtable** (`vec3::vtable::new`): the explicit path.
3. **Through the active table**, which can be located anywhere: the basis of driver polymorphism.

Calling a specific object's method requires `IX`. The method's address is taken from the object's vtable and loaded into `IY`, and then the call goes through it. The full form (with register saving, adding the offset, and jumping through `RET`) looks like this:

```
LD IX, $8000       ; object
PUSH HL / PUSH BC  ; save registers
LD L,(IX+0)        ; vtable address
LD H,(IX+1)
LD BC, $000F       ; method offset
ADD HL, BC
PUSH HL / POP IY   ; method into IY
POP BC / POP HL
PUSH IY            ; method address onto the stack
RET                ; "call" via RET
```

This is honestly expensive, and I labeled it as such: "with all the overhead." The `PUSH` + `RET` trick as an indirect jump is well known to anyone who has written for the Z80, and the cost shows why, in hot code, it's better to have a pre-built table and a single `JP (HL)` instead.

## 6. What Didn't Work Out

I left the most important part for the end. The project was not finished, and the reasons were both technical:

- **Compiler speed.** It was written on an interpreter, and the system came out complex and slow. For medium-sized projects this is tolerable, but scalability raised justified doubts in me.
- **Unfinished pieces.** Validation of method arguments (`parse-arg`) was not completed: I wanted the assembler to slap your hands if the wrong register or type was passed to a method. Instances (allocating memory for objects) and the static data compiler were not implemented, although all the introspection infrastructure for it was ready.
- **Porting the old drivers** to the new system never really began.

## 7. What Remains Useful

Even abandoned, the project suggests a few ideas worth taking with you:

1. **A single source of truth for structures.** A type description generates both the headers and the offsets for the assembler, as well as the descriptors. This cures the "hell of synchronization" better than any discipline.
2. **A vtable as a table of `JP`s.** Three bytes per method, a call in one jump, inheritance by copying.
3. **Polymorphism as a rejection of checks.** At 4 MHz the gain is not in the beauty of the architecture but in clock cycles: the mode is chosen once.
4. **Registers as part of the signature.** When types and registers are described together, a large class of errors disappears at build time.
5. **A meta-assembler on Lisp macros.** Macros provide everything for which other assemblers invent separate directive languages, and without separate languages.

If you still love the Z80, I hope these notes come in handy: not necessarily in Lisp, any macro assembler that can calculate offsets will do. The main thing is that an object model on an 8-bit machine really can be done, and it won't cost much.