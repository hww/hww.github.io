# Polymorphism in SOOT

## Table of Contents

- [Polymorphism in SOOT](#polymorphism-in-soot)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [The Idea: Dispatch Through the Type](#the-idea-dispatch-through-the-type)
  - [Methods on Types in SOOTC](#methods-on-types-in-sootc)
  - [The `basic` Type and Runtime Type Information](#the-basic-type-and-runtime-type-information)
  - [Why a SID and Not a Pointer](#why-a-sid-and-not-a-pointer)
  - [The `CallMethod` Instruction](#the-callmethod-instruction)
  - [Example: `(-> b len)` from Lisp to Assembly](#example---b-len-from-lisp-to-assembly)
  - [The Price and What We Got](#the-price-and-what-we-got)
  - [Conclusions](#conclusions)

---

## Introduction

In games, polymorphism is not needed for the beauty of hierarchies. The same call `(-> entity update)` must work for the player, for an NPC, for a door, and for a trigger. The code that makes the call does not know what is in front of it: the object itself knows.

A good design for this task is a simple one. To add a new type, you do not need to recompile the engine. To add a new method, you do not need to change the signatures of the existing ones. To change behavior, it is enough to replace the type's method table. All this fits into a few data structures and one virtual call instruction.

SOOTC (the SOOT compiler) implements such a scheme. This article explains why methods on structures do not give polymorphism by themselves, why the `basic` type is needed, why the object header holds a SID rather than a pointer, and why a separate `CallMethod` instruction had to be added for virtual calls.

> SOOT is an independent project. The approach is inspired by dispatch schemes publicly described for game scripting systems (including those in Naughty Dog's games and in the OpenGOAL project). The implementation and the data format are my own, and compatibility with those projects is not a goal. The names of games and projects belong to their respective rights holders.

---

## The Idea: Dispatch Through the Type

The scheme used by SOOTC works like this:

* every object that can take part in a polymorphic call has information about its type in its header;
* every type has a method table: a mapping `method_id → implementation`;
* a method call takes four steps: read the type from the header, find the table, take the implementation by `method_id`, and call it.

This is enough for the kind of polymorphism that matters in a game: the code does not know what object is in front of it, and the right implementation is found through the type.

---

## Methods on Types in SOOTC

SOOTC allows declaring methods on structures:

```lisp
(deftype my-basic (basic)
  ((x float) (y float) (z float) (w float))
  (:methods (len (my-basic) int)))

(defmethod len my-basic ((self my-basic)) 4)
```

At first glance this is OOP: there is a type, the type has a method, and the method is implemented separately. But **`deftype` by itself does not give polymorphism**. The reason is simple: for an ordinary structure, the compiler always knows the exact type of the value, and so it knows exactly which implementation to call. If you write:

```lisp
(deftype vec3 (structure) ((x float) (y float) (z float))
  (:methods (len (vec3) float)))

(defmethod len vec3 ((self vec3)) 3.0)
```

then `(-> v len)` for `v : vec3` is a **static** call to `vec3-len`. The compiler knows the exact type of `v` and the exact `method_id`, and generates a direct `LookupPointer` and `Call`. No table and no indirection. For `structure` types this is correct and fast.

Polymorphism begins where the compiler **does not know** the exact type: where `obj` is "something that has a method `len`," and what exactly it is will only be determined at run time. This needs a base type whose header holds information about the object's type.

---

## The `basic` Type and Runtime Type Information

SOOTC has a built-in type `basic`. All types that take part in a polymorphic call inherit from it:

```lisp
(deftype my-basic (basic) ((x float) (y float) (z float) (w float)) ...)
```

`basic` works like this: **the first 8 bytes of any object derived from `basic` are the identifier of its type at run time**. This is not a field declared by the user, but an implicit field inserted by the compiler. `TypeSystem` registers it at initialization:

```cpp
// basic has one implicit field: the runtime type tag.
add_field_to_type(basic_type, "type", make_typespec("sid64"));
```

The type `my-basic` inherits this field, and all its instances begin with 8 bytes holding the `sid64` of the type `my-basic`. The fields declared by the user (`x`, `y`, `z`, `w`) come **after** it:

```
my-basic layout:
  offset 0   : type (sid64, implicit)
  offset 8   : x (float)
  offset 12  : y (float)
  offset 16  : z (float)
  offset 20  : w (float)
  total size : 24
```

When an object is created:

```lisp
(define b (new 'heap 'my-basic :x 1.0 :y 2.0 :z 3.0 :w 4.0))
```

`NewNode::emit` for a type derived from `basic` first writes `SID("my-basic")` into the first 8 bytes and then initializes the fields:

```asm
; allocate("heap", "my-basic") -> r3
CallFf   r3, r0, 2
; r3[0..8] = SID my-basic
LoadStaticI64Imm r4, <sid my-basic>
StoreI64 r3, r4, 0
; r3[8..12] = 1.0
IAddImm  r5, r3, 8
LoadStaticFloatImm r6, <1.0>
StoreFloat r5, r6, 0
; ... y, z, w
```

Now any code that has a pointer to `b` can read `b[0..8]` and find out that this is a `my-basic`. This is the foundation of polymorphism.

---

## Why a SID and Not a Pointer

In the publicly described implementation of GOAL and OpenGOAL, the type information in an object's header is a pointer to the type structure. A pointer is faster: one read instead of two (reading the field and looking up the SID). But it has a weak spot: it points to a specific address in memory.

In SOOT, modules can be loaded, unloaded, and reloaded. If the object header held a pointer to `SsType`, then after a module was reloaded, the new `SsType` would sit at a different address, while all previously created objects would keep pointing to the old one. So the header holds a **SID** (`sid64`, the hash of the type name). It does not change on reload and does not depend on addresses. At call time the SID is turned into an `SsType*` through `Globals`:

```cpp
const sid64 type_sid = *static_cast<const sid64 *>(obj_ptr);
const StringId type_name(type_sid);
void *type_ptr = globals->find_symbol_ptr(type_name);
auto *type = static_cast<SsType *>(type_ptr);
```

This costs one extra read, but in exchange:

* a module can be unloaded and loaded again: the SID stays the same;
* a module can be updated: the new `SsType` with the same SID replaces the old one in `Globals`, and all objects created before the update see the new set of methods;
* method implementations can be changed without recreating objects.

The choice is: a bit slower but survives a module reload, instead of "fast but fragile."

---

## The `CallMethod` Instruction

We need an instruction that does three things:

1. takes a pointer to an object;
2. reads the SID from the header and turns it into an `SsType*`;
3. calls `SsType::m_pMethods[method_id].m_pLambda`.

`method_id` is known at compile time: the compiler sees `(-> obj len)` and knows that `len` is a method with `id = 9` on `my-basic` and on all its descendants. So `id` is a constant that can be put into a register.

The instruction has three fields (`a`, `b`, `c`). Field `a` is the result register, and field `c` is the number of arguments, including the receiver. That leaves one field, `b`, while a call has two inputs: the object and the `method_id`.

The calling convention removes this problem. The receiver, like the first argument of any call, lies in the first argument register (`r24`), so it does not need to be passed as a separate operand. Field `b` names the register that holds the `method_id`. The instruction format is not extended, and the range of the fields does not suffer. `method_id` lies in a register rather than in an immediate value because it can be large (up to several hundred).

---

## Example: `(-> b len)` from Lisp to Assembly

The user writes:

```lisp
(defmethod len my-basic ((self my-basic)) 4)
(define b (new 'heap 'my-basic :x 1.0 :y 2.0 :z 3.0 :w 4.0))
(-> b len)
```

**Step 1: `defmethod`.**

`defmethod len my-basic` is compiled into a `FunctionNode` named `my-basic-len`, with one parameter `self : my-basic` and the body `(return 4)`. `NodeBuilder::build_defmethod` registers the method in `TypeSystem`:

```cpp
MethodInfo info = m_ts.lookup_method(type_name, method_name);
m_ts.define_method(type_name, method_name, info.type, ...);
```

`info.id` for `len` is `9`: indices 0–8 are taken by the nine built-in methods of `basic` (`new`, `delete`, `print`, `inspect`, `length`, `asize-of`, `copy`, `relocate`, `mem-usage`). So `len` is `m_pMethods[9]`.

**Step 2: `define b`.**

`NewNode::emit` for `my-basic` generates the code we saw above: `allocate`, writing the type SID into `[0..8]`, and writing `x`, `y`, `z`, `w` into `[8..24]`. The result is a pointer in the heap, which is saved in the slot `b` through `StoreGlobalNode`:

```asm
LookupPointer r13, <SID b>
StorePointer  r13, r3, 0    ; slot_b = pointer to b
```

**Step 3: `(-> b len)`.**

`NodeBuilder::build_deref` sees `(-> b len)`, determines that `b` is a `my-basic` and `len` is a method with `id = 9`, and that `my-basic` is derived from `basic`. The check `should_use_virtual_methods(my-basic, 9)` returns `true`. A `MethodCallNode` is created with `m_method_id = 9`, and the object is `VariableNode b`.

`MethodCallNode::emit`:

```asm
; 1. receiver (b) in r24 as the first argument
LookupPointer     r0, <SID b>    ; slot_b
LoadPointer       r1, r0         ; b = *slot_b
Move              r24, r1        ; arg0 = receiver

; 2. method_id into a register
LoadStaticI64Imm  r2, <9>

; 3. virtual call: dst = r3, method_id in r2, argc = 1
CallMethod        r3, r2, 1

; 4. return the result
Return            r3
```

What `CallMethod` does in the VM:

```cpp
case Opcode::CallMethod: {
    // a = dst, b = register with method_id, c = argc
    // receiver = r24 (first argument, fixed offset)
    Variant &receiver = current_frame->get_argument(0);
    Variant &method_id_reg = current_frame->get_register(instr.b);

    // 1. Read the SID from the object header.
    const sid64 type_sid = *static_cast<const sid64 *>(receiver.get_ptr());

    // 2. Turn the SID into an SsType* through Globals.
    const StringId type_name(type_sid);
    auto *type = static_cast<SsType *>(globals->find_symbol_ptr(type_name));

    // 3. Take the implementation by method_id.
    i64 idx = method_id_reg.get_i64();
    ScriptLambda *lambda = static_cast<SsMethod *>(type->m_pMethods)[idx].m_pLambda;

    // 4. Call it.
    push_call_frame(lambda, instr.a, instr.c, mode);
    break;
}
```

**Step 4: `my-basic-len`.**

`FunctionNode my-basic-len` is compiled into:

```asm
Move              r0, r24          ; self = arg0
LoadStaticI64Imm  r1, <4>          ; return 4
Return            r1
```

The result is `4`. `CallMethod` puts it into `r3`, and `Return r3` returns the value to the caller.

---

## The Price and What We Got

The architecture is not perfect. The price is two additional accesses on every virtual call:

1. **reading the SID from the object header** (instead of a direct pointer to the method table);
2. **looking up the `SsType*` by SID in `Globals`** (a hash table).

I do not have exact measurements yet, so I will not quote numbers: a virtual call is certainly more expensive than a static one, but for game scripts this is acceptable. Scripts are called not in a tight loop but in per-tick logic, where the overhead is lost in the general background. For number-crunching tasks nobody forbids writing in a procedural style, for example `(defun my-len (v) ...)`, and then there is no price.

What we get in return:

* **polymorphism**: `(-> obj method)` works for any object derived from `basic`, regardless of the module in which it was created;
* **module reloading**: update the `SsType`, and all existing objects see the new set of methods (this is by design; some scenarios, for example redefining a method in another module, are not yet covered by tests, see the [project status](./project-status.md));
* **swapping implementations** without recreating objects;
* **predictability in time**: a call needs no allocations and no locks, and `method_id` is known at compile time, so there is no lookup by method name (the only lookup is by type name).

This is an honest engineering decision in the spirit of SOOT: simple, working, with a clear price.

---

## Conclusions

1. Polymorphism in SOOTC is not limited to "methods on structures." `deftype` by itself does not provide it: the compiler knows the exact type of a structure, so `structure` types use a static call.

2. Polymorphism begins with the `basic` type: its header holds the `sid64` of the type at run time, and the compiler can generate `CallMethod` without knowing the exact type of the object.

3. A SID instead of a pointer is a deliberate choice in favor of reloadable modules. The scheme described for GOAL and OpenGOAL gains one read through a pointer, while SOOTC uses a SID and gains the ability to update types on the fly.

4. `CallMethod` solves the shortage of operands with the calling convention: the receiver is always in the first argument register, and field `b` holds the register with the `method_id`.

5. The price: two additional accesses on every virtual call. For game scripts, this is an acceptable fee for polymorphism.

6. The solution is not ideal but honest: simple mechanisms, an explicit price, and no magic. And it works, which in engineering matters more than elegance.

---

[Project Status](./project-status.md)