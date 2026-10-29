# Modular SOOT System: Complete Guide

## Part 1: How Names Work in SOOT

In SOOT, two things must be distinguished: **how the compiler finds definitions and how the runtime stores their values.**

### 1. Definitions in Files

Each `.sc` file is a separate module. Within a single file, the same name cannot be declared twice:

```lisp
(defun greet () "hello")
(defun greet () "hi") ; error
```

To change a function, you must fix its definition, not add a second one.

At the same time, different files may define the same names.

### 2. How the Compiler Finds Definitions

When a file includes another via `(require "b.sc")`, the compiler gains access to its definitions.

If several files define `greet`, this is not an error: each definition is bound to its own file. When resolving a name, the compiler first checks the current file, then the included files.

Thus, **identical names in different files can exist independently of each other at the compilation level.**

### 3. What Happens During Execution

When a module is loaded, its global names are registered in the common `Globals` table.

In this table, each name corresponds to one current value. If several modules define `greet`, the last load replaces the previous value.

For example, `bar.sc` defines `greet` as `"hello"`, and `baz.sc` defines it as `"hi"`.

- If `bar` is loaded first, then `baz` — `Globals` will contain `"hi"`.
- If `baz` is loaded first, then `bar` — `Globals` will contain `"hello"`.

**Important:** this rule applies to the runtime. It does not mean that the compiler mixes definitions from different files.

### 4. What Happens in the REPL

The REPL is an interactive console. The compiler treats code entered in it as a separate virtual file, e.g. `<repl>` or `<repl-N>`.

Therefore, defining `greet` in the REPL does not delete the definition from `bar.sc` in the compiler's tables. But after loading a new definition, its value may replace the previous one in the common `Globals` table.

### 5. Summary

- **File:** contains its own definitions; re-declaring a name within a file is forbidden.
- **Compiler:** distinguishes definitions from different files and resolves names according to visibility rules.
- **Runtime:** stores one current value for each global name; the last load may replace it.
- **REPL:** creates separate definitions that, when loaded, may change values in the runtime.

---

## Part 2: Using the Command Line

### Compiling a Single File

```bash
sootc --no-repl --target build soot_src/tests/bar.sc
```

What happens:

1. The compiler reads `soot_src/tests/bar.sc`.

2. Builds an AST for it (`FileNode` with the name `bar`).

3. Generates `build/bar.bin` and `build/bar.h`.

4. Exits because of `--no-repl`.

**`--target build`** — the folder where results are written. If it doesn't exist, it is created. If not specified — defaults to `build`.

**What's inside `bar.bin`:**

- `bar-greet` — the compiled function.

- `top-level` — if the file contains top-level expressions (e.g. `(bar-greet)` without wrapping in `defun`). If the file contains only declarations, there is no `top-level` — the module does nothing on load, it only registers names.

### Compiling a File with Dependencies

```bash
sootc --no-repl --target build soot_src/tests/foo.sc
```

`foo.sc` contains:

```lisp
(require "bar.sc")

(defun foo-greet () (bar-greet))

(foo-greet)
```

What happens:

1. The compiler reads `foo.sc`.

2. Sees `(require "bar.sc")`.

3. Looks for `bar.sc` — first next to `foo.sc` (`soot_src/tests/bar.sc`), then in `PROJECT`, then in `search_paths`.

4. Finds it, compiles `bar.sc` as a separate module, registers it in the runtime.

5. Continues compiling `foo.sc`: `foo-greet` now sees `bar-greet`.

6. Generates `build/foo.bin`, which records: "this module requires `bar`".

7. Generates `build/bar.bin` (because `bar.sc` was also compiled).


### Compiling with SIDBase


```bash
sootc --no-repl --target build --sidbase build/sidbase.bin soot_src/tests/foo.sc
```

**SIDBase** is a file that maps 64-bit hashes (SIDs) back to strings. Inside `.bin`, all names are stored as SIDs (`0xF9E6E6EF197C2B25`), not as strings — this is more compact and faster. But without SIDBase, the disassembler and inspector show `<unknown:0x...>` instead of `bar-greet`.

**Why `--sidbase`:**

- Loaded at the start of compilation — so that all strings encountered in `lib.sc`, `lib.st`, and in your files get into SIDBase.

- Saved at the end — so that the disassembler can restore names.

**If `--sidbase` is not specified:**

- SIDBase is not loaded (empty).

- SIDBase is not saved.

- This is fine for one-off compilations, but `list-symbols`, `inspect`, and `sc-run` in the REPL will show `<unknown:...>`.

**Recommendation:** always pass `--sidbase build/sidbase.bin`. One file, accumulates names between runs.

### Starting the REPL with a Runtime

```bash
sootc --runtime local --sidbase build/sidbase.bin
```

- `--runtime local` — use the in-process VM (Carbon). Alternatives: `--runtime null` (execute nothing) or `--runtime host:port` (remote host).

- `--sidbase` — load SIDBase so that names resolve.

In the REPL you can:

- Compile forms (`(defun f () ...)`).

- Load modules (`(load-module "./build/foo")`).

- Call functions (`(sc-run foo-greet)`).

- Save the session (`(save-session "mysession")`).

- Save modules (`(save-module "soot_src/tests/bar.sc")`).

---

## Part 3: Using an Interactive Session

The compiler can be started using an interactive REPL session.

```bash
$ sootc --repl
```

The window that opens can be used for incremental compilation. To do this, enter an expression and press enter.

### What Happens When a Form Is Entered

For example, the user enters in the REPL:

```lisp
sci> (defun bar-greet () (print "hello"))
```

The following happens:

1. The loader parses the string into a SOOT object.

2. The REPL dispatcher checks: is this a REPL command (from `repl_commands()`) or an ordinary form? `defun` is not a command, so it goes further.

3. The compiler compiles the form as a separate module named `<repl-N>` (N grows with each input).

4. The runtime loads the module `<repl-N>`: `bar-greet` goes into `Globals`.

5. The runtime executes the `top-level` section of module `<repl-N>` (if present).

6. The form is remembered in `m_session_forms` — the session history.

**The main point:** each REPL input is a **separate module** `<repl-0>`, `<repl-1>`, etc. This is done so that redefining the same name in different inputs does not displace the old one in the compiler. In the runtime, `Globals` is one table — there the last one wins.

### The Problem of Erroneous Input

If you enter an incorrect `defun`, and any other incorrect expression, compilation fails, and the compiler state rolls back to the previous state:

```lisp
sci> (defun bar-greet (lambda () "wrong"))

; ERROR: FunctionCompiler::parse_arguments ...
```

This form will not get into `m_session_forms` — it is not part of the session. Only successfully compiled forms are recorded. Therefore `(save-session ...)` will not fail on old errors.

### Compiling a Dependency Request

```lisp
sci> (require "bar.sc")
```

This is a language form, not a REPL command. It is processed during compilation:

1. The compiler looks for `bar.sc` (next to the current file, then `PROJECT`, then `search_paths`).

2. Compiles `bar.sc` as a separate module (`FileNode("bar")`).

3. Registers the module in the runtime.

4. Remembers that the current file (or `<repl-N>`) requires `bar`.

**Result:** `bar-greet` is visible to the compiler in the current form. If in the next form you write `(bar-greet)`, the compiler will find it in `m_functions` (because `require` added it there via `compile_file`).

### Loading a Module in the Execution Environment

```lisp
sci> (load-module "./build/foo")
```

`load-module` is an ordinary function call that performs loading during program execution. It is not a compiler command and not a REPL dispatcher command. It:

1. Looks for `foo.bin` (first as is, then in `PROJECT`, then in `search_paths`).

2. Loads it into `Globals`.

3. **Recursively** reads `requires: ["bar"]` and loads `bar.bin`.

4. **Executes** the `top-level` section of module `foo.bin` (if present).

**The main difference between `require` and `load-module`:**

- `require "bar.sc"` — compiles `bar.sc`, registers it in the runtime, adds it to `m_functions`.

- `load-module "./build/bar"` — loads the already compiled `bar.bin` into the runtime, does not compile, does not add to `m_functions`.

**What follows from this:** after `(load-module "bar")` you can call `(sc-run bar-greet)`, but you cannot write `(bar-greet)` — the compiler does not know about `bar-greet`. For the compiler, `bar-greet` does not exist, because `bar.bin` did not go through `require`.

### Saving the Console History

```lisp
sci> (save-session "mysession")
```

What is saved:

- All forms that you entered in the console and that compiled successfully.

- Not saved: failed forms, runtime commands (`(e)`, `(reload)`, `(gc)`), forms from `require`.

What is not saved:

- Modules loaded via `load-module`. They are not in `m_session_forms`.

- `requires` from the console — because `load-module` is a runtime, not a `require`.

So, `mysession.bin` saves the forms entered in the console, not copies of all modules that the console loaded. If the saved session must have a compilation dependency on the source module `foo`, use `require`. `load-module` loads a ready binary module during execution, but does not create such a compiler dependency.

**Self-dependency:** if you save the session as `"bar"` and there was `(require "bar.sc")` in the console, `bar` will be removed from `requires` — otherwise `bar.bin` would require itself, and loading would be impossible.

### Saving a Module

```lisp
sci> (save-module "soot_src/tests/bar.sc")
```

What is saved:

- The contents of the file `bar.sc`: its `defun`, its `require`, its `top-level`.

- But not what you entered in the console.

**When this is needed:** when you have edited a `.sc` and want to compile exactly it without leaving the REPL. Or when you are working with several files and want to rebuild one of them.

**Self-dependency:** `save-module "bar.sc"` removes `bar` from `requires` — if `bar.sc` contains `(require "bar.sc")`, this is discarded.

### The Difference Between `save-session` and `save-module`

|                    | `save-session "x"`           | `save-module "x.sc"`                  |
|--------------------|------------------------------|---------------------------------------|
| What is saved      | forms entered in the console | contents of the file `x.sc`           |
| `requires`         | from `require` in the console| from `require` in the file            |
| `top-level`        | all console forms            | `top-level` from the file             |
| Self-dependency    | removed                      | removed                               |
| When to use        | "save what I've built up"    | "rebuild a file without leaving the REPL" |

---

## Part 4: Loading a Module

```lisp
sci> (load-module "./build/foo")

hello from bar

; => null
```

What happens:

1. `load-module` looks for `foo.bin` (first as is, then `PROJECT/foo.bin`, then `PROJECT/search_path/foo.bin`).

2. Loads `foo.bin` into `Globals`:

   - `foo-greet` — is registered.
   - `top-level` — is not registered (this is the module's entry point, not a symbol).
   - `requires` — is not registered (these are metadata).

3. Reads `requires: ["bar"]`.

4. Recursively loads `bar.bin`:

   - `bar-greet` — is registered.

5. Executes the `top-level` section of module `foo.bin`: `(foo-greet)` → `(bar-greet)` → `print "hello from bar"`.

6. Prints `hello from bar`.

**`; => null`** — `top-level` returns nothing, this is normal.

## Part 5: Calling a Function

```lisp
sci> (sc-run foo-greet)

hello from bar

; foo-greet => 1
```

`sc-run` is a REPL command. It:

1. Looks up the name `foo-greet` in `Globals`.

2. Finds `ScriptLambda*`.

3. Calls it via the VM.

4. Prints the result (`1` — because `print` returns `1`).

**The difference between `(sc-run foo-greet)` and `(foo-greet)`:**

- `(sc-run foo-greet)` — runtime, works because `foo-greet` is in `Globals`.

- `(foo-greet)` — compiler, will fail with `undefined function 'foo-greet'`, because `foo-greet` is not in `m_functions` (unless `foo.sc` was `require`-d from the console).

**If you want `(foo-greet)` to work:**

```lisp

sci> (require "foo.sc")   ; ← not load-module!

```

`require` compiles `foo.sc` and adds `foo-greet` to `m_functions`. After that, `(foo-greet)` compiles and executes.

---

## Part 6: Searching for Project Files

### Folders

- **`PROJECT`** — the project root. Determined by: `--project <dir>` → the directory of the first input file → `detect_project_root(cwd)`.

- **`build/`** — the default results folder (`--target build`).

- **`soot_src/`** — sources of `lib.st`, `lib.sc` (prelude).

### Files

- **`*.sc`** — source.

- **`*.bin`** — compiled module.

- **`*.h`** — C header with type descriptions (for external tools).

- **`sidbase.bin`** — SIDBase.

### Search paths

When searching for `require` and `load-module`, the following are checked:

1. **Next to the current file** (for `require`; for `load-module` — next to the `.bin`).

2. **`CWD`** — the current directory.

3. **`PROJECT`** — the project root.

4. **`PROJECT / search_path`** — `., scripts, src, examples, --source-root`.

---

## Part 7: Typical Scenarios

### Compile a Single File

```bash
sootc --no-repl --target build --sidbase build/sidbase.bin soot_src/tests/bar.sc
```

### Compile a File with Dependencies

```bash
sootc --no-repl --target build --sidbase build/sidbase.bin soot_src/tests/foo.sc
```

`bar.bin` will be created automatically (because `foo.sc` requires `bar.sc`).

### Open the REPL and Load a Module

```bash
sootc --runtime local --sidbase build/sidbase.bin

sci> (load-module "./build/foo")

hello from bar

sci> (sc-run foo-greet)

hello from bar
```

### Work in the REPL, Save the Session

```lisp
sci> (require "bar.sc")

sci> (defun my-fn () (bar-greet))

sci> (save-session "mysession")

; saved mysession.bin, requires: ["bar"]

sci> (e)
```

### Load a Saved Session

```bash

sootc --runtime local --sidbase build/sidbase.bin

sci> (load-module "./mysession")

hello from bar

sci> (sc-run my-fn)

hello from bar

```

### Recompile a File from the REPL

```lisp
sci> (save-module "soot_src/tests/bar.sc")

; saved bar.bin

sci> (load-module "./build/bar")

sci> (sc-run bar-greet)

hello from bar
```

---

## Part 8: Rules and Pitfalls

The following notes explain how the compiler works.

- **Each definition belongs to a file**
Within a file, a repeated name is an error. In different files — it's normal.

- **In the global runtime scope, the last one wins**
No matter how many files define `greet`. In `Globals` there is one `greet`, from the last loaded module.

- **The require expression is compilation, load-module is loading**
  - `require` → compiles `.sc`, registers in the runtime, adds to `m_functions`.
  - `load-module` → loads `.bin`, registers in the runtime, does not add to `m_functions`.
  It follows: after `require "foo.sc"` you can write `(foo-greet)`. After `(load-module "foo")` — only `(sc-run foo-greet)`.

- **The require directive searches next to the file**
  If `foo.sc` in `soot_src/tests/` requires `bar.sc`, `bar.sc` will be found in `soot_src/tests/`, even if `soot_src/tests/` is not in `search_paths`.

- **The REPL is a separate file**
  Everything entered in the console belongs to `<repl>`. It does not override files. In the runtime it replaces them.

- **The save-session directive differs from save-module**
  - `save-session` — saves the console.
  - `save-module` — saves the file.

- **A module cannot depend on itself**
  - `save-module "bar.sc"` removes `bar` from `requires`. `save-session "bar"` — too.

The following notes demonstrate potential problems.

- **Pitfall: The system does not check recursive dependencies**
  Although it is not correct to use them in principle, using an a->b->a dependency will not lead to a compilation error. It will not lead to an execution error either, but determinism may be lost.
- **Pitfall: `<unknown:0x...>` in `list-symbols`**
  If you started the REPL **without** `--sidbase`, names are shown as `<unknown:0x...>`. This is not a bug, it's the absence of SIDBase. Always pass `--sidbase`.
- **Pitfall: `foo.bin` in the project root**
  If you compiled `foo.sc` without `--target build`, `foo.bin` will end up in the `PROJECT` root. `load_module_recursive` searches for `foo.bin` first next to `foo.bin`, then in `PROJECT`, then in `search_paths`. If an old `foo.bin` is in the root — it may win over the new `build/foo.bin`. Always use `--target build`.

---

## Conclusion

To navigate the system, it is enough to remember three main distinctions: definitions inside source files and the runtime name table are not the same thing; `require` works with source code at compile time, while `load-module` loads a ready binary module during execution; `save-session` saves the console history, while `save-module` saves a specific source file.

- **File** — the unit of compilation. `defun`, `deftype`, `require`.

- **Compiler** — knows all files, but does not mix them.

- **Runtime** — one `Globals`, the last one wins.

`require` — compiles and registers. `load-module` — loads and registers. `save-session` — saves the console. `save-module` — saves the file.

Everything else is details.

---

[Project Status](./project-status.md)