# Incremental Builds Inside the Compiler: How sootc Decides What to Rebuild

Most build systems ask one question: *did the file change?* For sootc, that is the wrong question. This article explains why incremental compilation had to live inside the compiler, how a `.bin` file ends up carrying its own freshness manifest, and which trade-offs we accepted along the way.

## Contents

1. The problem
2. Why incrementality has to live inside the compiler
3. The `.bin` as its own manifest
4. Two views of dependencies
5. What makes a module dirty
6. Build modes, API levels and paths
7. In practice
8. Determinism
9. Build scripts written in soot
10. Why not separate compilation and linking?
11. What's left
12. Takeaways

## 1. The problem

Rebuilding a whole project is expensive, and the cost grows with the number of modules: more and more time goes into work whose result would not have changed.

External build systems (Make, CMake, Ninja) already know how to skip unchanged work. But they rely on two things only: **file modification times** and **explicitly declared dependencies**. They know nothing about what sootc does internally.

For C and C++ this is enough. `gcc` compiles one `.c` into one `.o`, and dependencies are expressed through `#include` and generated `.d` files. For sootc that model breaks down, and not because of a minor detail: it follows directly from how the compiler is built.

## 2. Why incrementality has to live inside the compiler

When sootc compiles `foo.sc`, it does much more than read one file:

1. It loads the prelude (`lib.st` + `lib.sc`) into the interpreter.
2. It loads `bar.sc` (and its dependencies) into memory.
3. It builds the node tree for `foo.sc` **in the context** of the dependencies already loaded.
4. Only then does it generate the `.bin`.

Now suppose CMake runs `sootc` once per file. Every process loads the bootstrap again and re-parses every dependency again. In a ten-module project that means ten bootstrap loads and ten parses of the same `bar.sc`. The savings from "not rebuilding what didn't change" are eaten by the cost of repeated startup.

So the idea is to make **the compiler itself** smart enough to decide what to rebuild, and to do it **in a single process** where the bootstrap and the dependencies are already loaded. An external build system can still drive things, but it launches `sootc` **once per project**, not once per file.

This is the reason sootc grew the following features:

- `--force` and `--incremental` build modes;
- `build-mode` in the config file, `SOOT_BUILD_MODE` in the environment, and `(opt "build-mode")` in the REPL;
- separate `out-dir` and `header-dir`, so build artifacts don't mix with sources;
- a `BuildInfo` record inside the `.bin`, so that the "dirty or fresh" decision is made from **content**, not mtimes.

## 3. The `.bin` as its own manifest

A `.bin` already contains a `requires` entry: the list of module names the file depends on. The loader needs it, and it reads the names as plain **strings** straight from the string pool, without needing `SIDBase`. This keeps the `.bin` self-contained.

We added a second entry, `build-info`. It is a **compiler feature, not a loader feature**: the runtime never reads it. Its only job is to let sootc answer one question: *does this `.bin` still match its source and its dependencies?*

The layout:

```
BuildInfo               (0x20)
DepEntry[num_deps]      (0x10 * num_deps)
```

```cpp
struct BuildInfo {
    u64 source_hash;       // FNV-1a 64 of the .sc source
    u64 compiler_version;  // hash of the compiler version
    u64 platform_hash;     // FNV-1a 64 of "linux"/"windows"/"darwin"
    u32 num_deps;
    u32 reserved;
};

struct DepEntry {
    sid64 name_sid;        // SID of the module name (without .sc)
    u64   bin_hash;        // FNV-1a 64 of the dependency's .bin
};
```

The key decision: **`DepEntry` stores only the `sid64` of the name and the hash of the dependency's `.bin`.** The name strings already live in `requires`. Duplicating them in `build-info` would cost extra bytes and, worse, create one more place where the two copies can drift apart.

## 4. Two views of dependencies

Dependencies are described twice, and that is deliberate:

- **`requires`** holds strings (`const char*` in the string pool). The loader needs them, and they work without `SIDBase`.
- **`build-info.deps`** holds `sid64` + `bin_hash`. The compiler needs them, and they only work where a `SIDBase` exists, which for sootc is always.

The freshness check, `is_fresh`, walks **from `requires` to `build-info`**, never the other way around:

1. Take `requires.m_pNames[i]`, which is guaranteed to be a real string.
2. Compute `SID(name)`, a deterministic CRC64.
3. Look up the entry in `build-info.deps` with that `sid64`.
4. Take its `bin_hash`, build the `.bin` path from the name string, hash that file, and compare.

The reverse direction (`StringIdManager(d.name_sid)` → string) is unreliable. `SIDBase` may not know the string, `get_string` then returns an empty string, and the check silently fails. Making `requires` hold strings is precisely what prevents this.

## 5. What makes a module dirty

A module is dirty if **any** of the following holds:

- **`source_hash` differs.** The `.sc` file changed.
- **A dependency's `bin_hash` differs.** The dependency's `.bin` changed (or is missing).
- **The `.bin` is missing.** There is nothing to verify, so it must be built.
- **`compiler_version` differs.** The compiler was upgraded.
- **`platform_hash` differs.** The target platform changed.

For now `compiler_version` is just a version string (`"sootc-v1"`). Hashing the contents of `lib.sc` / `lib.st` into it is the next step. The field already exists in the format, so the `.bin` layout won't need to change.

## 6. Build modes, API levels and paths

**Modes**

- `--force` rebuilds everything and ignores `build-info`.
- `--incremental` rebuilds only what is dirty.

The mode can be set through the CLI, the environment (`SOOT_BUILD_MODE`), JSON config (`"build-mode"`), or the REPL: `(opt "build-mode" "incremental")`.

**Two API levels**

- **`save-module`** is the explicit "save this module" command. It always writes the `.bin` and the `.h`, regardless of `is_fresh`. If you asked for it, you get it.
- **`save-module-deps`** is the project-level build. It respects `build_mode`: in `Incremental` it skips fresh modules (both the root and its dependencies) and rebuilds dirty ones.

The CLI follows the same logic: `sootc --incremental foo.sc` skips the input file if it is fresh.

**Paths.** `project_root` is resolved relative to the directory of `soot.json` (if the path is relative), or relative to the CWD otherwise. `out-dir` and `header-dir` follow **the same rule**: they resolve from `json_dir`, not from `project_root`. This matters because build artifacts live in their own directories and don't have to sit inside the source tree. Values coming from the environment or the CLI resolve from `project_root`, since they don't know about `json_dir`.

## 7. In practice

**Explicit command on a fresh module:**

```
sc> (save-module "tests/foo")
; saved soot_gen/bin/tests/foo.bin
; OK: module 'tests/foo' saved

sc> (save-module "tests/foo")   ← second time
; saved soot_gen/bin/tests/foo.bin
; OK: module 'tests/foo' saved
```

`save-module` writes every time. It is an explicit command.

**Project build, everything fresh:**

```
sc> (save-module-deps "tests/foo")
; OK: module 'tests/foo' and dependencies saved   ← nothing written, all fresh
```

**Dirty root, fresh dependency:**

```
sc> (save-module-deps "tests/foo")
; saved soot_gen/bin/tests/foo.bin                 ← only foo
; OK: module 'tests/foo' and dependencies saved
```

**`bar.bin` was deleted:**

```
sc> (save-module-deps "tests/foo")
; saved soot_gen/bin/tests/bar.bin                 ← bar was missing
; saved soot_gen/bin/tests/foo.bin                 ← foo is dirty (dependency changed)
; OK: module 'tests/foo' and dependencies saved
```

Note how the second rebuild follows from the first: once `bar.bin` is regenerated, its hash no longer matches the one recorded in `foo`'s `build-info`, so `foo` is rebuilt too.

**From the command line:**

```bash
sootc --incremental soot_src/tests/foo.sc
# everything fresh: "Skipping soot_src/tests/foo.sc: up to date"

sootc --force soot_src/tests/foo.sc
# always compiles
```

## 8. Determinism

An incremental build must not change the result. The test is simple: a clean build (`--force`) and an incremental build after a series of edits must produce **byte-for-byte identical** `.bin` files.

This works because string IDs are derived from CRC64 of the string, so the order of compilation doesn't matter. Auto-generated lambda names (`<base>::lambda#N`) are local to their module and don't leak into the result.

If the test ever breaks, the cause will be one of three things: ordering, hidden inputs (the prelude, the compiler version), or `BuildInfo` itself. So far it passes.

## 9. Build scripts written in soot

sootc has a `--do <script>` option: it runs a script in the compiler session **before** the input files. That makes it possible to describe a build **in soot itself** instead of in shell:

```scheme
; build.sc
(require "tests/foo.sc")
(require "tests/bar.sc")
(save-module-deps "tests/foo")
(save-module-deps "tests/bar")
```

Run it with:

```bash
sootc --incremental --do build.sc
```

The compiler loads the prelude and the dependencies once, runs `save-module-deps` for each module, and in `Incremental` mode rebuilds only what is dirty. It is the same incremental logic as `save-module-deps` alone, but the project build is now expressed as data rather than as CLI flags.

This is what makes a project build possible **without an external build system**. CMake can still invoke `sootc --do build.sc` as a single rule, but the logic of *what* to build and *in which order* lives in `build.sc`.

## 10. Why not separate compilation and linking?

In theory, the "proper" design would be to compile every file **independently** and link afterwards. Incrementality then becomes trivial: a file changed, so rebuild that one file and let the linker assemble the rest.

We didn't go that way, for three reasons:

1. **It is much more work.** You need a real linker, symbol resolution, an ABI, and initialization ordering. sootc has none of that: modules are loaded at runtime by name.
2. **Linker errors are the hardest to debug.** They have the weakest connection to anything in the source. "Symbol not found" is nothing like "you forgot a parenthesis on line 42". They are rarer, but when they happen they are painful.
3. **In sootc, "linking" is loading a module into the VM.** Modules are loaded and compiled separately, and that is a feature, not a bug: it is what enables interactive programming, the REPL, and a dynamic environment. The feeling that "the module loaded, so it works" is part of the design.

File-by-file compilation is therefore **possible**, and already works: `sootc foo.sc` compiles one file. It is just **inefficient** for a project, because every invocation reloads the prelude and the dependencies. That is why incrementality lives **inside** one compiler process, where both are already loaded.

A separate brute-force mode doesn't need to be built, since it already exists. The value lies in letting **one process** build **many files** intelligently, and that is what `save-module-deps`, `--do build.sc` and `--incremental` do.

## 11. What's left

- **Hash `lib.sc` / `lib.st` into `compiler_version`.** Today it is a version string. The field is already in the format, so switching to a content hash won't change the `.bin` layout.
- **Pruning orphaned `.bin` files.** If a module is deleted, its `.bin` stays behind. We need either a `--prune` command or automatic cleanup during incremental builds.
- **A depfile for CMake.** Something like `sootc --incremental --depfile=foo.d foo.sc`, so CMake knows what `foo.bin` depends on. Right now CMake can run `sootc --do build.sc` as a single rule, which is enough, but a depfile would make the integration more precise.
- **Lock in the "clean == incremental" test.** It already passes; it should be pinned down in CI.

## 12. Takeaways

The solution is not ideal. The ideal one is independent compilation plus linking, but that is far more work, and linker errors are harder to diagnose. Besides, sootc is by nature a dynamic environment where modules are loaded and compiled separately, and that is a design choice rather than a shortcoming.

What we built is a compromise:

- **The `.bin` is self-contained.** `requires` serves the loader, `build-info` serves the compiler. No external manifest is needed.
- **`is_fresh` compares content hashes, not mtimes.** Moving the project, changing permissions or running `git checkout` doesn't break incrementality.
- **Two modes and two API levels.** `--force` / `--incremental` and `save-module` / `save-module-deps` give explicit behavior for explicit commands, and incrementality for project builds.
- **`--do build.sc`** lets you describe a build in soot itself, with no shell scripts.
- **Determinism.** `--force` and `--incremental` produce byte-for-byte identical `.bin` files.

It is **simple**: no separate manifest, no fingerprint files, and (for now) no depfile. It is **practical**: incrementality pays off most in projects where loading the prelude takes longer than compiling a single module. And it **fits the spirit of soot**: data (`.bin`, `build.sc`) instead of external tooling, explicit commands instead of magic.

Not perfect, but it works, and it is arguably the best trade-off for a compiler that lives in a REPL and builds a project from the inside.
