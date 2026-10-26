# Variant: The Universal Data Container of the Virtual Machine

## Table of Contents

- [Variant: The Universal Data Container of the Virtual Machine](#variant-the-universal-data-container-of-the-virtual-machine)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [A Container Built on a Union](#a-container-built-on-a-union)
  - [Which Types to Support](#which-types-to-support)
  - [Surrogate Types: The Idea and Why I Dropped It](#surrogate-types-the-idea-and-why-i-dropped-it)
  - [Three Register Types](#three-register-types)
  - [How to Show Values in the Terminal](#how-to-show-values-in-the-terminal)
  - [Pointers and the Garbage Collector](#pointers-and-the-garbage-collector)
  - [Strict Getters and Soft Converters](#strict-getters-and-soft-converters)
  - [Arguments of Native Functions](#arguments-of-native-functions)
  - [A Possible Extension: The basic Type](#a-possible-extension-the-basic-type)
  - [Good to Know](#good-to-know)
  - [Conclusions](#conclusions)

---

## Introduction

The SOOT virtual machine uses fixed-size stack frames. Inside a frame lie virtual registers, and each register is a `Variant` container that can store a value of one of the supported types. This article explains why it is built the way it is and why it has so few types.

---

## A Container Built on a Union

The natural solution for a register is a `union`: a small piece of memory in which an integer, a floating-point number, and a pointer take turns living. But a `union` by itself does not know what it currently holds. So next to it there must be a field that stores the type of the value.

`Variant` is exactly such a pair: a one-byte type tag plus an anonymous `union` of three alternatives (`i64`, `f64`, and `void *`). With alignment, the container takes 16 bytes on a 64-bit platform. The class serves as a wrapper: it encapsulates constructors, checks, and conversions so that the rest of the code does not work with the `union` directly. The same could be done without a class, but with it the code is easier to read.

The main question lies elsewhere: which types the container should know about.

---

## Which Types to Support

At first I wanted to store the type as a CRC32 or CRC64 of its name. Then a pointer could carry the type of the object it points to, and the set of types would be open: a new type could be added without changing the container itself. The idea seemed elegant, but in practice it turned out to be a dead end.

Every operation on the container selects its implementation by type: for example, to turn a value into a string or into another type, you must call exactly the code that knows that type. With a small dense set of tags, this is solved by an ordinary `switch`. With an open set of hashes, a `switch` is impossible, and every operation has to look up the right method in a table. For the registers of a virtual machine, the hottest spot in the system, that is too expensive.

For a while the system worked exactly this way. After going down this path, I reduced the number of types to a minimum. But even the minimum had to be defined deliberately.

---

## Surrogate Types: The Idea and Why I Dropped It

At the next stage, the container kept types that are needed not by computations but by people: a boolean, a string identifier (`StringId`), and a string. The virtual machine has no instructions for them. These are "surrogate" types: they derive from an integer or a pointer and exist for convenience.

The idea is not a silly one. When a script returns a value to the terminal, `true` and `false` read better than `1` and `0`, a string reads better than a memory address, and the original string reads better than a 64-bit hash. A tag in the container provided this information for free.

But it has a flaw: **the instruction set does not guarantee that the tag will be set correctly in a hundred cases out of a hundred.** The `String` tag is set when a string is loaded from the constant area. But once you store it in a structure field and read it back with an ordinary read instruction, the register again holds a plain pointer: the instruction does not know that the field holds a string. The result of a comparison instruction remains an ordinary integer, so it becomes a boolean only where native code explicitly created one. A string identifier obtained as the result of a computation remains simply an integer.

The upshot is that the type is set in some cases and not set in others. Such a half-measure is worse than having no type at all, because it is misleading: looking at a value without a tag, nobody knows whether it is a boolean, and looking at a value with a tag, you think you can rely on it.

This could be fixed in two ways. The first: add an instruction to the processor for "set the register's type," so that a value is loaded first and then assigned a type. This brings dynamic typing back in pieces: tags have to be maintained everywhere, and the code gets more complex. The second: multiply the instructions, so that each one that can put a string or a boolean into a register has a separate variant. The instruction set grows, and the virtual machine becomes more complex. Both paths contradict the main idea of the project: a simple solution where a complex one does not pay off.

So I decided to drop the surrogate types and keep in the container only what is really supported by instructions.

---

## Three Register Types

Currently `Variant` distinguishes three types (`RuntimeType`):

| Type | What is stored | Note |
| --- | --- | --- |
| `Int` | signed 64-bit integer | covers booleans, characters (`char`), and `StringId` |
| `Float` | 64-bit floating-point number | `f32` values are widened on write |
| `Pointer` | a pointer to anything | the object's type is not stored; an empty value is a null pointer |

The principle is this: **the type lives in the code, not in the data.** A register does not know that it holds a boolean or a string identifier. The instructions and the compiler know that. When a function checks a value for truth, it checks an integer. When a `StringId` is passed to a native function, the function interprets the integer as an identifier according to its signature. The same goes for a pointer to a string.

This is also how processors themselves and many virtual machines work: registers are not labeled, and the correctness of usage is ensured by the compiler.

---

## How to Show Values in the Terminal

Dropping the surrogate types hurts debugging: instead of `true`, the terminal now shows `1`. Most of this loss can be recovered on the tooling side without complicating the virtual machine.

- **Static types from the compiler.** The language is strictly typed, and the compiler knows the type of every register at every point of a function. This information can be written to an optional debug table (register, instruction range, type) that is not included in release builds. In the REPL it is even simpler: the compiler knows the type of the expression it has just compiled, and the result can immediately be printed as `true` rather than `1`. The types of native function arguments are already described in their signatures.
- **Guesses for values without a type.** If there is no type (for example, when viewing raw registers), the terminal can show a value together with interpretations: `1 (bool: true)`. For `StringId` this is reliable: the value is either found in the table of string identifiers or it is not, and an accidental match of a 64-bit hash is practically impossible.
- **Strings.** A pointer that lies in the module's string area, or in a heap block whose header carries the string type, can be shown as a string without adding a tag to the register.

These mechanisms live in the development tools. They cost the virtual machine nothing.

---

## Pointers and the Garbage Collector

The `Pointer` type can point to anything, and the container itself does not know whether the object is managed by the garbage collector. For this, as described in the article on memory, the manager has a simple predicate: it checks whether the address falls within the single heap region.

`Variant` does not own memory. Its destructor is empty, and copying is shallow: after a copy, two values point to the same object. The garbage collector manages the lifetime of objects.

This gives a rule for whoever stores `Variant` values, primarily stack frames. During marking, a frame must walk its registers and call `mark_object` for every pointer. There is no need to tell "own" pointers from "foreign" ones: `mark_object` itself ignores addresses outside the heap. If a register is skipped, the collector will free the object it points to.

---

## Strict Getters and Soft Converters

`Variant` has two families of methods for accessing a value.

**Strict getters** (`get_i64`, `get_f64`, `get_ptr`, and others) return the value only if the type matches. Otherwise they throw `TypeError` stating the expected and actual types. This is a defensive check: it catches programmer errors in native code.

**Soft converters** (`to_int`, `to_float`, `to_bool`, `to_string`) try to bring the value to the required form. An integer becomes a floating-point number, a pointer becomes a boolean (non-null or null), and any value becomes a string for output. `to_string` is convenient for debugging: any register can be printed without knowing its type in advance, with a pointer shown as a hexadecimal address. The `fmt` formatter can do the same, so values read well in logs right away.

---

## Arguments of Native Functions

The registers of the virtual processor lie in a frame one after another. This makes it possible to pass arguments to native functions without intermediate containers: a pointer to the first argument register and the number of arguments is enough. A native function receives a variadic list of `Variant` values and converts them to the types it needs.

Normally there should be no errors here. A native function has a declared signature, and the compiler will not generate a call that passes an argument of the wrong type. The strict getters remain as a fallback safeguard. The function's context always determines how to read a value: if a register holds an integer while the function expects a string identifier, it treats the number as a `StringId`; if a pointer to a string is expected, it reads the pointer.

In this model, strings have become a "native" object: checking length, comparing, and concatenating are done by native functions. For game logic this is acceptable, because a game does little with strings. A side effect is that strings can no longer be compared by content with `==` on `Variant`. `==` compares pointers.

---

## A Possible Extension: The basic Type

For future extensions, one more type could be added. The type system has a base type for RTTI (runtime type information), called `basic`. Its difference from an ordinary structure is that its first field stores the identifier of the object's type. The idea and the type tree are inspired by the publicly described type system of GOAL and OpenGOAL.

This gives two capabilities:

- **Access to fields by name.** It is useful for debug output, logging, and a kind of reflection in C++ code. Ordinary access to fields from C++ works without it, because script structures are compiled into a C++ header. But type information makes it possible to look up fields by name.
- **Runtime polymorphism.** Knowing an object's type, you can call methods by index. In theory this makes it possible to write a visitor for any type, even one that was not known at compile time.

How much either capability is needed in a real game is still unclear, but the potential is there. If it turns out to be needed, a pointer to a `basic` object would become a kind of `Pointer`, not a new register type.

---

## Good to Know

- **`to_c_string()`** returns a pointer to an internal per-thread buffer. The next call overwrites the previous result, so it can only be used immediately.
- **`Pointer` does not store the object's type.** What lies at the pointer is determined by context.
- **A boolean is an integer.** Anything that is not zero counts as true.

---

## Conclusions

I consider `Variant` the optimal solution for virtual registers. The code is simple and readable, the `switch` is tiny, and the data is easy to inspect in a debugger. The same container works well for passing arguments to native functions.

The main design principle: the type lives in the code, not in the data. The container holds only what the processor instructions actually handle. A new type should enter the list only if it is truly necessary and only if the instruction set can guarantee its correct value. Everything else, that is, a convenient presentation for humans, lives on the side of the development tools.

There is one more observation: keeping so few types in a value is possible only with strict compilation. All conversions and type checks are performed by the compiler, and the runtime keeps a tag only for protection. In a dynamic interpreted language, this would not work. SOOT itself is a good example: its interpreter has about twenty object types, while the virtual machine's registers have three.

---

[Project Status](./project-status.md)

---