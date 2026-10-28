# The Global Scope: Why SOOT Has No Private Names

## Table of Contents

- [The Global Scope: Why SOOT Has No Private Names](#the-global-scope-why-soot-has-no-private-names)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [What Is Customary in General-Purpose Languages](#what-is-customary-in-general-purpose-languages)
  - [What This Would Mean for the Virtual Machine](#what-this-would-mean-for-the-virtual-machine)
  - [The Solution: One Global Scope](#the-solution-one-global-scope)
  - [Anonymous Entities Today](#anonymous-entities-today)
  - [The Price of the Decision](#the-price-of-the-decision)
  - [What Next: Private Definitions](#what-next-private-definitions)
    - [How This Will Affect the File Format](#how-this-will-affect-the-file-format)
  - [Conclusions](#conclusions)

---

## Introduction

The article about the REPL already says that in an interactive session every module exports everything defined in it. Over time this decision spread to the whole language: SOOT has no private names, and the definitions of all modules go into a single global scope. This article describes why I decided this, what it gives, and what it costs.

---

## What Is Customary in General-Purpose Languages

The usual rule for general-purpose languages is this: a module hides what belongs to its internals and exports only what is meant for others. Private functions and data cannot be called from outside, and names inside different modules do not conflict. For large programs with many authors this is the right decision: it protects internal details and allows them to be changed without breaking the rest of the code.

To work this way, every definition has to be marked as exported or not. This leads to a flag in the record and to two scopes: the file scope and the common, global one.

---

## What This Would Mean for the Virtual Machine

In SOOT, a function call is simple. The `call` instruction takes the function from a register, and the function gets into the register in only one way: by a lookup by name. So every access to a function, method, or structure turns into a lookup by name, and this lookup must be fast.

Now imagine private names. Exported definitions can be indexed in a common table. But the names of private definitions may coincide with the private names of other modules, so they cannot be put into the common table. That leaves three options:

- search for a private name by scanning the file itself, which is slow;
- start a second table keyed by "module plus name";
- give each module its own hash table of the required size.

Any of them produces a multi-level lookup: first the file scope, then the global one. Tables, file format, and loading all become more complicated, and every access gets more expensive. All this for the sake of encapsulation, which game scripts hardly need: scripts are written by a small team, and there is usually no need to protect the internal functions of one script from another.

---

## The Solution: One Global Scope

I refused these complications. There is no export flag and there are no private names; the language has one global scope. Any entity of any module is found by name with a single lookup.

This is handled by the `Globals` class. It is as simple as the decision itself: a flat hash table from a string identifier to a `Symbol` record:

- `ptr` is the address of the definition's slot inside the loaded module's memory. It is not the value itself, but the place where the value lies;
- `typeId` is the type of the definition;
- `ownerModulePath` is the module the definition belongs to.

`Globals` does not store values and does not own files: the module registry owns them. It only answers the question "in which loaded module and at what address does the entity with this name lie?" The address is valid while the module stays loaded.

The result: one hash table lookup per access, no second tables, and no extra structures in memory. It is important not to overstate the gain: I have not tried to measure the difference in numbers, but one lookup instead of two and the absence of additional tables are in any case simpler and cheaper.

---

## Anonymous Entities Today

There is a case that looks like an exception: structures without a name. If a new structure is created when initializing a pointer field, the compiler gives it a generated name. Nobody will look this structure up by name, but in the current scheme it also ends up in `Globals`.

This gives an unexpected bonus for debugging. Any such structure can be found in the table, and you can see where it lies in memory and what fields it has. For tools, this is a free way to look inside a module.

In the end, the principle "everything is indexed" turned out to be consistent: a generated name is no different from any other.

---

## The Price of the Decision

A single scope has a price, and it is better to name it directly.

- **Name collisions.** If two modules define the same name, the one loaded later wins: the entry in `Globals` is simply overwritten. For the REPL this is exactly what is needed (a new definition displaces the old one), but for ordinary modules it can come as a surprise. A natural improvement is a warning at load time if the name is already taken.
- **No real encapsulation.** A script can accidentally redefine a library function or call something the module's author considered internal.
- **Unloading a module.** Its names disappear from the table, and the definitions it shadowed do not come back. As long as modules are not unloaded, this is not a problem.

All this can be compensated by a naming culture: prefixes by subsystem, distinguishable names for library and user functions. For game scripts written by a small team this is an acceptable price. For a general-purpose language such a decision would be bad, and I do not offer it as a model. For the narrow niche SOOT was created for, it honestly fits.

---

## What Next: Private Definitions

A single scope does not close the door on private entities. The idea is that today's `define` is replaced by an explicit choice: a definition can be public or private to the module (`define private`). For a private entity, the compiler generates code that looks it up not by name but loads the definition's address directly. The name is resolved at compile time rather than at run time, so at run time such an entity is essentially anonymous and has no business in `Globals`. The two-level scope does not come back: lookup by name remains the only way to find a public entity, while a private one is not looked up by name by anyone.

Anonymous initializer structures fall into the same category: they are referenced only by pointer.

### How This Will Affect the File Format

The simplest solution, a "do not index" flag in every record, does not fit: all records in the file are aligned to 64 bits, and one bit of flag would cost an extra 8 bytes per record. So the list of definitions in the file is split into two parts: public ones first, then private ones. The file header stores a single number: how many definitions are public. At load time, the module registers in `Globals` only the first part. The format hardly becomes more complicated, and no memory is spent on flags.

The names of private definitions can be kept in the file for tools: a debugger will find them by walking the module, while the common table stays clean.

I do not promise any dates: for now everything is indexed, and the system works great that way.

---

## Conclusions

Module isolation is good for general-purpose languages, but every private entity requires a separate level of lookup. For a virtual machine in which every call begins with a lookup by name, this is expensive and complex. So SOOT has a single global scope, and the problems that arise are solved by naming discipline.

The decision is not very clean, but it is simple and honest, and in this it resembles the rest of SOOT's architecture. If private definitions are ever needed, the file format is ready to accept them almost unchanged: it is enough to split the list of definitions into public and private ones and write their number into the header.

---

[Project Status](./project-status.md)
