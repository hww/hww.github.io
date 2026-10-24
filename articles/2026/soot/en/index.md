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

More articles will be added here as they are published.

## Status

SOOT is a young project and is still under active development. The repository was recently selected for inclusion in a public code index for AI agents.

## Contact

I'm currently looking for full-time opportunities in Compiler Development, Language Tooling, and Systems Programming. If you work in those areas, I'd be glad to connect.