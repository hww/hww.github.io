# What Already Works in SOOT

This is a snapshot of the project's state at the commit `fix: virtual method dispatch for basic-derived types`. It lists only what works and has been verified in the REPL or in tests. There are no plans or promises here.

The goal of this document is to show honestly what can already be picked up and used, and what cannot yet.

---

## The Language and the Interpreter (Script)

The interpreter works on top of an S-expression reader.

* **Reading code.** Symbols, numbers (integers and floats), strings, keywords, `#t` and `#f`, quote, lists, and `#foo` as a string identifier (SID) literal.
* **Basic forms.** `define`, `lambda`, `if`, `when`, `unless`, `cond`, `let`, `set!`, `quote`, `begin`.
* **Macros.** `defmacro` works both in the interpreter (`lib.st`) and in the compiler (`lib.sc`). These are two independent sets of macros: the function `try_getting_macro_from_soot` looks only in the `*soot-env*` environment.
* **Functions.** Calling by name, recursion, and closures (in the interpreter).
* **Loops.** `while` with an optional `:label`.
* **Input and output.** `_print`, `_format`, and string handling.

The interpreter is used in the REPL (`--interpret-only`, `(sc-eval ...)`), and the compiler is built on top of it.

---

## The Compiler (Sootc)

The compiler translates SOOT forms into bytecode for the Carbon virtual machine.

* **Functions.** `defun`, `lambda`, `defmethod`. Parameters can be written with a type, `(x int)`, or without one, in which case they are `object`. Returning values, constants, local variables, and temporary registers all work.
* **Structures.** `deftype` with fields and inheritance (from `basic` or `structure`). The options `:methods`, `:size-assert`, `:method-count-assert`, `:heap-base`, and `:pack-me` are supported.
* **Enumerations.** `defenum`.
* **Constants.** `defconstant`: the value is substituted at compile time.
* **Native functions.** `define-extern` and `define-native` with a `(function ...)` signature. They are called with the `CallFf` instruction.
* **Static data.** `(define name (new 'static Type ...))`, including nested anonymous structures.
* **Heap data.** `(define name (new 'heap Type ...))`.
* **Arrays inside data.** `(new 'static 'inline-array T N ...)` and `(new 'heap 'inline-array T N ...)`. Element access: `(at arr i)`.
* **Method calls.** `(-> obj method)`. For types derived from `basic`, the call is polymorphic, through `CallMethod` (see `docs/polymorphism.md` for details). For types derived from `structure`, the call is static, through `LookupPointer` and `Call`.
* **StateScript.** `deftype` with `:states`. The handlers `:enter`, `:exit`, `:code`, `:trans`, `:post`, `:event`, and tracks inside states.
* **`require`.** Includes a file using the search paths. A file is never loaded twice.
* **`seval`.** Evaluates a form with the interpreter at compile time.
* **`update-macro-metadata`.** A stub for tools: it does nothing.

---

## The Virtual Machine (Carbon)

* **Processor.** A register machine with 34 registers: `r0`–`r23` for local values and `r24`–`r33` for arguments. It has arithmetic, comparisons, bitwise operations, jumps, loads and stores, calls (`Call`, `CallFf`, `CallMethod`), `LookupPointer`, and `LoadStatic*`.
* **Memory.** Arenas and a garbage collector that walks objects using type descriptions (`TypeRegistry`). Pointers inside structures are described through `PtrField`, and special cases can supply their own traversal function (`mark`). From the REPL, `gc::collect()`, `(gc)`, and `(gcls)` are available.
* **Processes.** A scheduler (`Scheduler`) with `spawn`, `tick`, and `run_loop`. A native function can yield control to other processes (`should_yield`).
* **States and tracks.** `StateScript` is stored as data: `SsState`, `SsOnBlock`, `SsTrackGroup`, `SsTrack`, `SsLambda`. One state can run several tracks at once, and each track controls its own aspect of the animation.
* **Modules.** Loading (`load_module`, `load_module_from_binary`, `load_module_with_imports`), unloading (`unload_module`), and reloading (`reload_module`). The module registry (`ModuleRegistry`) stores `ss_type_sids` for rollback.
* **Global symbols.** `Globals` is a flat "SID → pointer" table. You can register a symbol (`register_symbol`), find it (`find_symbol_ptr`), and remove all symbols of a module (`remove_module_symbols`).

---

## The Binary Format (.bin)

* **File layout.** A `DC_Header` header, a `DCEntry` table of records, the data, a string pool, and a relocation table (a bitmap with one bit per 8 bytes).
* **Relocation.** A slot stores an offset in the file, and the corresponding bit is set in the map. At load time `read_reloc_table` adds the file's address in memory to the offset, which gives an absolute pointer. The mechanism works for functions, strings, and pointers to static data, and has not been changed so far.
* **Sections.** `ScriptLambda`, `SsType`, `StateScript`, `DataStruct`, `Array`, `Map`.
* **`SsType`.** An array of fields `SsField[]` and an array of methods `SsMethod[]`. The pointer to a method's lambda (`m_pLambda`) is stored as an offset in the file and is relocated.
* **`StateScript`.** `SsOptions`, `SsState[]`, `SsOnBlock[]`, `SsTrackGroup`, `SsTrack[]`, `SsLambda[]`.
* **Loading.** `BinaryFile::from_path` and `BinaryFile::from_buffer` read a file, `read_reloc_table` applies relocations, and `get_unmapped` returns the data in its original form for saving.

---

## The REPL (Sootc)

* **Commands.** `(help)`, `(clear)`, `(e)`, `(reload)`, `(sizes)`, `(inspect-env)`, `(inspect-env-soots)`, `(has-macro "name")`, `(list-symbols)`, `(save-session)`, `(sc-run name args...)`, `(sc-eval expr)`, `(arg-set "key" value)`, `(arg-get "key")`, `(arg-ls)`, `(gc)`, `(gcls)`, `(ts-info name)`, `(ast)`.
* **Compiler parameters.** `print-listing`, `print-ir`, `print-ast`, `print-asm`, `log-level`. Read-only: `sidbase`, `runtime`, `user-profile`.
* **Input convenience.** History, hints, autocompletion, and highlighting (the replxx library).
* **Incremental work.** Each input gets its own name, `<repl-N>`. If `deftype` and `defmethod` are entered separately, they are still assembled into one structure (`owners_to_rebuild`).
* **Rollback on error.** The states of `FileNode` and `GlobalNode` are saved as a snapshot (`Snapshot`). If compilation fails, the changes are rolled back and the session stays usable.

---

## What Does Not Work Yet, or Works Only Partially

This section is as important as the previous ones: without it, the list above turns into advertising.

* **Method redefinition.** `defmethod` for an already defined method replaces the old `FunctionNode` in the structure and rebuilds it. But this is not covered by tests and has not been verified in every case (another module, a native method).
* **Checking that methods are complete.** `deftype` with `:methods` does not require all declared methods to be implemented. `save_binary` prints a warning, and the compiler does not stop. A call `(-> obj method)` for an unimplemented method crashes at run time with `UndefinedFunction`.
* **`build_ss_type_for_script`.** The pointer `SsMethod::m_pLambda` is written through `push_value_with_ptr`, so the offset will be counted twice at load time. As long as `StateScript` is not tested, the bug is invisible, but it is marked.
* **Polymorphism across modules.** `require` loads a module but does not check whether its methods are compatible with the current `deftype`.
* **Macros in `defmethod`.** Macro expansion inside a method body works, but there are no tests.
* **`StateScript` serialization.** The code exists, but the full scenario (create a state-script, run it, wait for a track) has not been verified.

---

## Summary

SOOT can read S-expressions, interpret them, and compile them to bytecode. It loads bytecode into the virtual machine and executes functions, works with structures and enumerations, calls methods statically and polymorphically, builds and loads `StateScript` with states and tracks, starts processes and schedules their execution, collects garbage, and saves and loads `.bin` files with relocations.

SOOT cannot yet check that a declared method is implemented, and cannot check the contract when methods are redefined. It does not work with `StateScript` end to end, and it does not check types across modules.

This is a young project. The list above is a starting point.