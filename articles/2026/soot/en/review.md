# SOOT Architecture Overview

## Table of Contents

- [SOOT Architecture Overview](#soot-architecture-overview)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Origins](#origins)
  - [The Type System and the Interpreter](#the-type-system-and-the-interpreter)
  - [Metaprogramming](#metaprogramming)
  - [The Virtual Machine](#the-virtual-machine)
    - [How the Virtual Machine Works](#how-the-virtual-machine-works)
  - [File Format](#file-format)
  - [Summary](#summary)
  - [Project Status](#project-status)

---

## Introduction

This article is an overview of SOOT's architecture: how the project arrived at its current shape, what parts it consists of (the type system, the interpreter, the compiler, the virtual machine, the file format), and what decisions were made along the way. Individual subsystems are covered in more detail in other articles in the series.

The name SOOT reflects the character of the project. It has developed unevenly and is far from the polish of the systems that inspired it: OpenGOAL and Naughty Dog's scripting systems. In that sense it can be called dirty, but without a negative connotation. It is honest, though: it is an attempt to build something of my own, by my own principles, studying other people's solutions and open code without carrying someone else's project over wholesale. Hence the name: the work is dirty, but honest. The virtual machine is called Carbon, to tell it apart from the compiler. The names are related in meaning: soot consists mostly of carbon.

> **Project independence.** SOOT is an independent project. It is not affiliated with Naughty Dog, Sony Interactive Entertainment, or the authors of OpenGOAL. It is neither a copy nor a fork of someone else's project: the architecture and implementation were developed by me, and the ideas are based on publicly available materials (talks, articles, documentation) and on studying OpenGOAL's open-source code. In some places, fragments of OpenGOAL code, which is distributed as an open-source project, are used. The names of games and projects belong to their respective rights holders and are mentioned only to explain sources of inspiration. Direct compatibility with those projects is not a goal of SOOT.

---

## Origins

The project started with the idea of building a compiler for game development. By then I had already studied a significant amount of publicly available material about scripting systems in Naughty Dog's games. That material indicated that the studio's earlier games used a Lisp-like language with its own approach to compiling game code, while in later projects scripts ran on a virtual machine that gave control over execution.

The project was started several times in different programming languages. Each time, once there was some result, it ran into another major refactoring, and I didn't have the energy and time for it, so the work came to a complete stop.

---

## The Type System and the Interpreter

A separate stage was the type system. It is a low-level library that stores metadata about types and inheritance. Its core principles are consistent with the approaches publicly described for GOAL and OpenGOAL, but the implementation is my own.

I wrote the type system several times: in Racket, Common Lisp, and Go. The Go version turned out to be the most successful from a practical standpoint and was used in commercial projects. It differs noticeably from the other versions because it is designed for network communication between a Unity project and a Linux backend.

After this work, I was able to draw a few conclusions:

- Programming in Common Lisp turned out to be the most comfortable, and development time in it was the shortest.
- Go came very close to Common Lisp in comfort and development pace.
- Working in Racket turned out to be unsuccessful.

Some time later, after analyzing the results, I decided to write a Lisp interpreter from scratch. I wanted to walk this path on my own and understand how everything works, relying only on public descriptions. That is why the scripting Lisp in SOOT is my own implementation, and it differs noticeably from other known implementations. Debugging was fairly difficult in places.

After that, I rewrote the Go version in C++, and it moved closer to the type system the compiler needs.

The result was a type system and an interpreter.

---

## Metaprogramming

The next step was adding metaprogramming features to the interpreter: access to the AST and to source code positions, and the ability to create binary files and link them. This produced a metacompiler implemented on top of the interpreter.

After that, the project was frozen again. Some time later, metaprogramming was removed from the interpreter, and the compiler was rewritten in C++. The interpreter and the type system were kept in the form in which they had already been implemented.

---

## The Virtual Machine

This is where SOOT diverges noticeably from OpenGOAL. As far as public descriptions show, the GOAL and OpenGOAL compiler produces processor code, that is, machine code for specific platforms. SOOT generates code for a virtual machine, for three reasons:

1. It is simpler.
2. Naughty Dog's public talks indicate that in the studio's later projects scripts ran on a virtual machine, and this option seemed the most attractive to me.
3. Virtual machine code is easier to translate into code for a specific processor later.

A virtual machine is easier to control: the execution of a process can be suspended and resumed later. This capability, execution control with elements of cooperative multitasking, looked the most attractive, and the compiler was designed around such a machine. This influenced the semantics and syntax of the Lisp-like language.

SOOT's virtual machine is inspired by the approach described in public materials about The Last of Us (TLOU) scripting system, but it is an independent development. Reproducing that system was never the goal, so the differences are substantial. As far as public descriptions allow me to judge, they are the following:

- TLOU uses purely procedural programming, while SOOT also has object-oriented programming.
- In TLOU, data (structures) and executable logic (state-scripts, that is, scripts describing finite state machines) are clearly separated. State-script declarations are organized in a dynamic style, as a list of names and values, and fields are accessed through the native functions `get` and `set`. Native functions extend the language and are written in C++, for example `print` and `format`. In SOOT, state-scripts are compiled into structures, and fields are accessed by offsets.

### How the Virtual Machine Works

The opcode takes one byte. The remaining 24 bits are used for operands (two or three, depending on the instruction), so all instructions have a fixed size of 32 bits. This is a three-operand virtual machine: its instructions work with virtual registers located in a stack frame.

- Each `StackFrame` has a pointer to its parent, and the frames form a tree.
- A `StackFrame` has a fixed size, so it is convenient to allocate it from an object pool.
- Each `StackFrame` stores the execution state of the virtual machine, so it serves as both a closure and a continuation.

---

## File Format

A file consists of a header and a list of definitions, each of which has a name and a type. The structure of the format conceptually resembles the binary script files that public materials describe for Naughty Dog's games, but the SOOT format is its own and is not designed to be compatible with them.

In addition, the file stores information about every type: field names, offsets and sizes of structures, and the `vtable` and `states` tables. It is too early to say whether this is a good decision. It did turn out to be simpler: by that time, compilation of structures and their methods already existed, and state-script became one of the variants of structures.

Names and type names are represented in the file as CRC64 values, and the corresponding strings are saved in a separate sidbase file. Definitions can include lambda functions, types, finite state machines, structure instances, and other entities. At the end of the file are string literals and unnamed structure instances. Local literals are saved at the end of each lambda function.

---

## Summary

SOOT developed fairly chaotically, without a plan known in advance. Decisions were made along the way, and many elements were rewritten several times, changing each time. Even more code went into the archive, and some decisions were simply removed.

The SOOT compiler is adapted to its own virtual machine and to running that machine's code inside a game engine. This approach gives control over code execution and leaves open the possibility of translating the code into code for a specific processor in the future, either at runtime or through static cross-compilation.

In addition, the SOOT compiler combines object-oriented programming with high-level constructs such as state-script for describing game logic. The language is oriented toward animation programming.

---

## Project Status

The project is not finished and is under development.