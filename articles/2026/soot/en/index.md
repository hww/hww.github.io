# SOOT

**A compiler and virtual machine for a strictly typed language with interactive development**

**English** | [Русский](../ru/index.md)

---

## About the Project

I've been building a compiler and a runtime VM for a while now. Mostly on my own, and mostly because I wanted to understand how these complex systems fit together from first principles.

The project has grown into much more than a compiler. It now includes:

- a garbage collector;
- an incremental REPL;
- a scheduler that runs scripted state machines at a fixed 60 Hz tick.

## Origins

The compiler itself grew out of code from OpenGOAL (the open-source Jak & Daxter project). The VM design is based on my own research: I analyzed the game's script bytecode formats using custom Python tooling and combined these insights with the studio's public talks and documentation.

I'm not aiming for direct binary compatibility. Where the original implementation was unclear or undocumented, I designed my own architectural solutions.

## The Questions Behind It

Every single detail demands an explicit decision, and even the smallest nuance compounds the overall system complexity. For example:

- How should a REPL compile a new expression in the context of everything entered before it?
- What are the trade-offs of garbage collection in a real-time runtime?
- What does "frame 10" mean: the 10th tick or the 11th?

None of these are trivial, and each answer directly shapes how the entire engine behaves.

## Article Series

In this series I share the engineering decisions behind the project: the alternatives I considered, the reasoning that led to each solution, and the trade-offs that came with it.

1. [SOOT Architecture Overview](./review.md)
   The story of how SOOT was developed and an overview of its architecture: the path from several restarts in different languages to a type system, an interpreter, and a C++ compiler; why the compiler targets a virtual machine rather than a processor; how the VM works (32-bit instructions, a tree of `StackFrame`s) and the binary file format.
2. [Incremental Interactive Compilation: Combining a Strict Compiler with a REPL](./incremental-repl.md)
   How the REPL uses the same compiler that builds the project: a module per input, a global name table, `top-level` initialization, and a deliberate decision not to unload modules.
3. [Memory Management in SOOT: From Reference Counting to a Simple Garbage Collector](./memory-manager.md)
   How the memory model evolved: from passing values and reference counting through per-frame data to a simple non-moving garbage collector; type descriptors that keep the memory manager independent of the VM, arenas and size classes, a pass number instead of clearing marks, incremental sweep, known limitations, and plans for improvement.
4. [Anatomy of the SOOT Interpreter: Three Engineering Decisions](./script-debug.md)
   How the interpreter helps track down errors: `TextDb` links S-expressions to places in the source code, `vararg_check` describes the argument contract of built-in functions in one line, and trace accumulation with error handlers written in the language itself gives controlled execution; where these mechanisms come from in GOOS and what they cost.
5. [Variant: The Universal Data Container of the Virtual Machine](./variant.md)
   How the registers of the virtual machine are built: a tag plus a union of three types (integer, float, pointer); why storing the type as a CRC turned out to be a dead end; why the surrogate types (boolean, `StringId`, string) had to be dropped, since the instruction set cannot guarantee that they are set correctly; how to show values in the terminal on the tooling side; the connection to the garbage collector and how arguments are passed to native functions.
6. [The Global Scope: Why SOOT Has No Private Names](./globals.md)
   Why every definition of every module lives in a single global scope: what module isolation would cost a virtual machine where every call starts with a lookup by name, how the `Globals` index works, what the decision costs (name collisions, no real encapsulation) and how naming discipline compensates for it, and the planned `define private` with a file format that splits definitions into public and private parts.
7. [Polymorphism in SOOT](./polymorphism.md)
   How the same call `(-> obj method)` works for different kinds of objects: why methods on structures do not give polymorphism by themselves, why the `basic` type carries a type identifier in its header, why that is a SID and not a pointer (to allow module reloading), how the `CallMethod` instruction deals with the shortage of operands, and what all this costs; with a walkthrough of an example from Lisp to assembly.
8. [Modular SOOT System: Complete Guide](./modules.md)
   How the compiler and runtime handle names, how `require` differs from `load-module`, what `save-session` and `save-module` save, working with SIDBase, and common pitfalls.

More articles will be added here as they are published.

## Status

SOOT is a young project and is still under active development. The repository was recently selected for inclusion in a public code index for AI agents.

## Contact

I'm currently looking for full-time opportunities in Compiler Development, Language Tooling, and Systems Programming. If you work in those areas, I'd be glad to connect.