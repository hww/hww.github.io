# Incremental Interactive Compilation: Combining a Strict Compiler with a REPL

## Table of Contents

- [Incremental Interactive Compilation: Combining a Strict Compiler with a REPL](#incremental-interactive-compilation-combining-a-strict-compiler-with-a-repl)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [A Strict Compiler with an Interactive REPL](#a-strict-compiler-with-an-interactive-repl)
  - [Every Input Is a New Module](#every-input-is-a-new-module)
  - [How a New Module Sees Everything That Came Before](#how-a-new-module-sees-everything-that-came-before)
    - [What Happens on Redefinition](#what-happens-on-redefinition)
  - [Module Naming in the REPL](#module-naming-in-the-repl)
  - [Saving a Session](#saving-a-session)
  - [Module Initialization at Runtime](#module-initialization-at-runtime)
    - [Why `top-level` Is Separate](#why-top-level-is-separate)
  - [How a Module with Dependencies Is Loaded](#how-a-module-with-dependencies-is-loaded)
    - [What This Gives Us](#what-this-gives-us)
  - [Conclusion](#conclusion)

---

## Introduction

The SOOT compiler was designed for the game industry. In game development, interactive and incremental workflows pay off substantially: the "change, run, see" cycle shrinks from minutes to seconds, and in large projects from hours to minutes. The shorter the cycle, the more experiments a team can run in a day.

But game development is about more than speed; it is also about division of labor. In large studios, core code (engine, physics, rendering, networking) is written by engineers who need a strict, statically typed compiler: types catch errors before the program runs, optimizations work from known types, and the binary is built AOT (Ahead-Of-Time) and needs no interpreter in production. Game designers, scripters, and content designers, on the other hand, write logic on top of the core: events, states, dialogue, NPC behavior. They need an interpreter or a REPL where they can type an expression, see the result immediately, change a rule, and not rebuild the project.

Languages usually pick one of the two: either a strict compiler (and game designers suffer) or a dynamic interpreter (and the core loses speed and reliability). SOOT tries to combine both modes in one language: strict typing and AOT compilation for the core, a REPL and interactive development for the logic. This article describes how that combination is built.

---

## A Strict Compiler with an Interactive REPL

The naive approach to combining them goes like this: the REPL interprets, the compiler compiles, and they somehow stay in sync. At first glance this works. The REPL is an interpreter, so why would it need a compiler? The compiler is for building the project, so why would it need a REPL? Let each do its own job.

The phrase "somehow stay in sync" only describes the problem; it offers no solution. Synchronizing an interpreter with a compiler is exactly the work that has to be done, and all the complexity lives there. The naive approach has three concrete problems: two semantics (the interpreter and the compiler understand `if`, `when`, and `not` differently), two name tables (the interpreter keeps names in its own environment, the compiler keeps them in the VM's `Globals`), and two type systems (the compiler requires explicit types, the REPL does not). Each has to be solved separately, and each drags several more problems behind it.

In SOOT's architecture, the REPL does not interpret code. It calls the compiler, the same one that builds `.sc` files into `.bin` files. Each input becomes a new module: it is parsed, compiled, and loaded into the runtime. From the compiler's point of view, there is no difference between the file `game.sc` and a line typed into the terminal: both take the same path, `AST → FileNode → BinaryFile → load_module`.

The REPL turns out to be a thin wrapper. All the complexity (types, macros, code generation) lives inside the compiler, and the REPL only calls it and loads the result. The three problems above do not arise here: there is no second semantics, no second name table, and no second set of types. There is one compiler, one `Globals`, one `TypeSystem`. The REPL uses the same path, only with a shorter entry.

There is an interpreter in this scheme too, but its role is different: it expands macros before compilation. This is an important detail, and it deserves a separate write-up.

---

## Every Input Is a New Module

Working in a REPL means working in a session. The programmer starts it, enters expressions one after another, and expects everything entered to persist: definitions, variables, and functions stay available to later inputs until the end of the session. This is the basic property of an interactive environment; without it, a REPL is useless.

The simplest solution that comes to mind is to compile everything accumulated so far into a single module and replace the old module with the new one on every input. But then all the code runs again on every input. You enter `(print "hello")` and `hello` is printed. You enter `(print "world")`, the first `print` runs again, and the console shows `hello`, `hello`, `world`. The REPL turns into "restart from the beginning," and the session loses its meaning.

The ideal option is to compile each expression into a separate module and load it into the runtime without touching the previous ones. Then `(print "hello")` runs once, `(print "world")` also runs once, and the console shows `hello` and `world`.

One could go further and build an elaborate concatenation system, where a new module is "appended" to the old one and the two are merged into one. But that requires tracking what changed, what stayed, and what was redefined. For a REPL, this complexity is not justified.

Instead, a simple decision was made: every input is a new module. It is compiled and loaded into the runtime, and the global name table (the one that indexes all definitions from all loaded modules) is updated. The definitions of the new module take effect, and the previous modules stay in memory.

---

## How a New Module Sees Everything That Came Before

Every input is a separate module, but from the programmer's point of view it is one session and one scope: if I entered `(defun f ...)` and a line later `(f 5)`, I expect `f` to be visible. Modules are the compiler's internal machinery, and the programmer does not think about them. For the programmer there is one REPL, one set of definitions, one session.

The compiler sees a different picture. For it, each module is an isolated unit with its own names, its own table, and its own definitions. By default a module exports nothing: everything defined in it stays internal. Ordinary `.sc` files work this way: `(defun f ...)` in `a.sc` is not visible in `b.sc` until `a.sc` explicitly exports `f`.

This creates a contradiction: the programmer expects a shared scope, while the compiler provides module isolation. The two have to be reconciled without breaking either.

The solution is simple: in the REPL, everything is exported. Every module created from an input is compiled with the flag `export_all = true`. This flag tells the compiler to export everything defined in the module. The definitions are registered in the global name table (`Globals`) as if this were an ordinary module that had explicitly exported each of its definitions.

From there, the module-loading machinery takes over. When a new module is loaded into the runtime, `Globals` is extended with its exports. The next input is compiled after that, and when `NodeBuilder` looks up a name, it finds it in `Globals`, on equal footing with names from `lib.sc`, from `(require ...)`, and from previous inputs. No special synchronization between modules is needed: `Globals` is a single table, and it already sees everything that has been loaded.

This is what makes the REPL transparent. The programmer enters `(defun f ...)`, then `(f 5)`, and `f` is found. The reason is that `f` sits in `Globals` like any other export; the REPL does not need to "remember" it. From the compiler's point of view at the second input, `f` is an external entity from another module; from the programmer's point of view, it is their own, from the same session. Both are right, because `Globals` does not distinguish between sources.

### What Happens on Redefinition

Each module keeps its own name table. When the module is loaded, this table is indexed into the global one: each name from the module is recorded in `Globals` and points to the definition in that module. If such a name is already in `Globals`, the entry is switched to the new module.

Suppose `X` is defined in module A, and then module B, which also contains `X`, is loaded. From that moment `Globals` points to `X` from B. Names that B does not redefine keep pointing to A. The definition from the module loaded later wins, so the load order is the order of priority.

This means that at any moment `Globals` is an up-to-date snapshot of the whole session: all loaded modules are in effect, and for each name the most recent definition is selected.

---

## Module Naming in the REPL

Each module is loaded under a unique name: `<repl-0>`, `<repl-1>`, `<repl-2>`. This is a condition of correctness. If all modules had the same name, `load_module` would unload the previous module whenever a new one was loaded, and the names would disappear from `Globals`. A unique name guarantees that a new module supplements the old ones and does not replace them.

There is a second reason, related to heap variables. A module can have static fields that refer to dynamic values, for example `(define x (new 'heap ...))`. `top-level` creates such a value on the heap and writes a reference to it into the module's static field. If the module were unloaded, the field would disappear, and the garbage collector would reclaim all objects that nothing refers to anymore. As long as the module stays in memory, the entire context it created is preserved.

The price of this decision is memory growth. Modules are not unloaded until the end of the session: every input adds a new one, and after a hundred inputs there are a hundred modules in memory. All of them remain in effect, but for each name `Globals` points to the most recent definition. Old definitions that have been redefined are no longer used by anyone, yet they still occupy memory.

One could call this a leak, but the word is too strong. A leak occurs when memory is lost in an uncontrolled way and cannot be recovered because nothing refers to it anymore. The situation here is different: the memory is held by modules that the compiler still references, and it will be released when the session ends. It is more accurate to call this retention. A leak is a bug; retention is a deliberate decision.

Why is it justified? Modules in the REPL are small. One input is a few lines of code, a few definitions, a few kilobytes of binary. Even a thousand inputs amount to a few megabytes. For a person at a terminal this is imperceptible. For a server that runs for months it would be noticeable, but a REPL lives only as long as its session, and a session is one day of a programmer's work.

Most importantly, the alternative is more expensive. To unload old modules, you must know for certain that nothing refers to them: not `Globals`, not other modules, not closures, not saved pointers. In an interactive session this cannot be tracked: the programmer may have saved a function, called it from a new input, or built a closure. Any mistake leads to a dangling pointer and a crash. The complexity is out of proportion to a gain of a few megabytes.

So modules are not removed. Each input supplements the previous ones, `Globals` points to the most recent definitions, and old modules stay in memory until the end of the session. The simplicity of this decision is what makes the REPL reliable: there is no reference graph to track, no fear of unloading too much, and no need to explain to the programmer why their function "suddenly disappeared."

---

## Saving a Session

While a session is running, the compiler remembers everything that was entered. Each input becomes a module, and the module is loaded into the runtime, but the source forms (what the programmer typed in the terminal) and the trees built from them remain in the compiler's memory. The runtime does not need them, since it works with binaries. The compiler needs them so that the whole session can be rebuilt at any moment.

The `save-session` command does exactly that: it takes all the forms entered during the session, builds a single `FileNode` from them, compiles it into a single `BinaryFile`, and saves it to disk as `.bin` and `.h`, like an ordinary compiled module. From the compiler's point of view, there is no difference between a session of a hundred inputs and a file `game.sc` of a hundred forms: both are a set of forms from which a module is built.

This is convenient. A session in which you debugged code, redefined functions, and experimented turns into a reproducible binary. It is a rebuild from the source forms, with no memory dump or state snapshot. If you run this `.bin` tomorrow, it will execute the same code you entered, in the same order. This is useful for debugging: here is the session in which the bug was found, and here is the binary that reproduces it.

However, `save-session` is not a way of programming; it is a supporting tool. Writing code in the REPL, saving it to a `.bin`, and treating the result as "source" is a bad idea: a `.bin` is unreadable, cannot be diffed, cannot be reviewed, and cannot be merged. It is best to write code in `.sc` files, which are compiled the usual way, go into version control, and go through code review. The REPL is for experiments, and files are for code. `save-session` is the bridge between them: "I experimented, it worked, now I'll move it into a file."

In short, `save-session` is convenient and useful, but it is not a methodology and does not replace the normal development cycle.

---

## Module Initialization at Runtime

A module contains two kinds of code. The first is declarations: `defun`, `deftype`, `defenum`, `defmethod`. They are not executed at load time: they register names and types so that the code that will be executed can find them. The second is expressions: `(print "hello")`, `(set! x 5)`, `(define a (new 'heap 'acc))`. They must run exactly once, in the order in which they are written, when the module is loaded.

The compiler collects all top-level expressions into a single function and calls it `top-level`. This function is synthetic; the programmer did not write it. Its body is a `begin` of all the expressions that did not go into declarations. If there are no expressions (the file contains only `deftype` and `defun`), `top-level` is not created, because there is nothing to execute.

An example. Suppose we have this code:

```sc
(define a (new 'heap 'acc))
(print "started")
(set! a.total 10)
```

`(define a (new 'heap 'acc))` is both a declaration and a runtime action. In the declaration, `a` gets the type `acc` and a slot in the file. In the runtime action, a heap object is created and written into the slot. The first part goes into the module's declarations, and the second goes into `top-level`. `(print "started")` and `(set! a.total 10)` also go into `top-level`. As a result, `top-level` looks roughly like this:

```
top-level:
    a = new 'heap 'acc
    print "started"
    a.total = 10
```

This code is generated by the compiler and performs what the programmer wrote outside of functions.

### Why `top-level` Is Separate

Loading and execution are different things. The module loader must not execute code: it must register the module in the runtime (names, types, binary). Executing `top-level` is a separate step, and it is done explicitly.

This matters because some modules are libraries. `lib.sc` contains `defmacro`, `deftype`, and `defmethod`, and it contains no expressions that need to be executed at load time. `lib.sc` has no `top-level`, and the loader does not run it. If loading and execution were a single step, `lib.sc` would be executed on every load, which is pointless and potentially dangerous (macros should not "fire" just because a library was loaded).

That is why `load_module` registers and `run_top_level` executes: two operations, two explicit steps.

---

## How a Module with Dependencies Is Loaded

A module can depend on other modules: through `(require "x.sc")` in the source, through a `.dci` manifest with imports, or through an explicit load from the host. The loader works recursively: it first loads the dependencies, then the module itself.

For each module, the order is:

1. Load its dependencies (each by the same rules).
2. Load the module itself.
3. Execute its `top-level`, if it has one.

For a module `A` that depends on `B` and `C`, where `B` depends on `D`, it looks like this:

1. Load `D`, execute `D.top-level`.
2. Load `B`, execute `B.top-level`.
3. Load `C`, execute `C.top-level`.
4. Load `A`, execute `A.top-level`.
5. Return to whoever requested the load of `A`.

The key point is that `top-level` is executed immediately after its module is loaded, before the next module is loaded. By the time `A` is loaded, `B` and `C` are already loaded and their `top-level` has already run. If `A` uses `B.some_function`, that function is already defined and initialized.

The order is interleaved: load, `top-level`, next load, `top-level`. This order guarantees that by the time `A.top-level` runs, all its dependencies have gone through both loading and initialization. If all `top-level` functions were executed after all loads, `A.top-level` could reach for a `B` that is loaded but not yet initialized and get wrong data.

### What This Gives Us

- Libraries (`lib.sc`, `(require ...)`) are loaded but not executed: they have no `top-level`, and `run_top_level` does nothing.
- The main module is loaded and executed: its `top-level` plays the role of `main`.
- A REPL input is loaded and executed: the programmer entered an expression, and it must run.
- Dependencies are loaded and executed before the module that depends on them: by the time `A.top-level` runs, all of them are initialized.

This is why `top-level` is a separate function and not part of loading. It serves as the module's entry point, like `main` in C, only it is created automatically from the top-level expressions.

This also explains how the REPL works. Each input is a new module with its own `top-level`: the expressions the programmer entered that did not go into declarations. `(print "hello")` in the terminal is a top-level expression, and it goes into the `top-level` of the module `<repl-N>`. `(defun f ...)` is a declaration, and it goes into the module's declarations. `(define a (new 'heap 'acc))` is both: the declaration of `a` and a runtime action in `top-level`.

When an input is loaded, `run_top_level` runs the `top-level` of that module, and `print` is executed exactly once. The next input is a new module with a new `top-level`, and the old one is left untouched. That is why `(print "hello")` and `(print "world")` in two inputs produce `hello` and `world`, rather than `hello`, `hello`, `world`: each input has its own `top-level`, and it runs once.

This is the same mechanism that ordinary modules use. The REPL follows the same path, "load, then `run_top_level`," only with a shorter entry and a unique module name.

---

## Conclusion

We have ended up with a language that is usually considered inherently contradictory: strictly typed, with mandatory type declarations and AOT compilation, yet with a simple syntax accessible to a game designer and with interactive development that does not require rebuilding the project for a single changed line. As a rule, these properties exclude each other: typing requires a compiler, interactivity requires an interpreter, and combining them head-on does not work.

They were combined through architecture. The REPL does not interpret code; it calls the same compiler that builds `.sc` into `.bin`. Each input becomes a new module, `Globals` sees everything that has been loaded, and `top-level` runs once after loading. Each of these decisions can be explained in a single sentence.

The main takeaway is something else: a task that is complex in the general case can be made simple by a handful of simple technical decisions. Incremental compilation, module unloading, and synchronizing two name tables each turn into a serious subsystem on their own. In SOOT, each of them is replaced by a simple decision: modules are not unloaded, there is one name table, and a REPL session needs no dependency graph. On the one hand, these are limitations. On the other, they deliver a sufficient result at a small cost and make the system reliable, because there are simply fewer places where it can go wrong. We made each simplification deliberately, weighing complexity against benefit.

This approach has a price: memory in the REPL grows with the number of inputs, and there is no incremental recompilation. The price is predictable and bounded, and for the tasks the language was created for, it is justified. SOOT does not aim to be a universal language. It is a language for a specific niche, where the core is written with strict typing, the logic with simple syntax, and development happens interactively.

---

[Project Status](./project-status.md)

