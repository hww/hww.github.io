# Memory Management in SOOT: From Reference Counting to a Simple Garbage Collector

## Table of Contents

- [Memory Management in SOOT: From Reference Counting to a Simple Garbage Collector](#memory-management-in-soot-from-reference-counting-to-a-simple-garbage-collector)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Early Versions](#early-versions)
    - [Values Instead of Pointers](#values-instead-of-pointers)
    - [Reference Counting](#reference-counting)
    - [Per-Frame Data](#per-frame-data)
    - [The Garbage Collector](#the-garbage-collector)
  - [How the Memory Manager Stays Independent of the VM](#how-the-memory-manager-stays-independent-of-the-vm)
  - [How the Garbage Collector Works](#how-the-garbage-collector-works)
    - [Overall Design](#overall-design)
    - [Memory: the Region and Arenas](#memory-the-region-and-arenas)
    - [The Object Header](#the-object-header)
    - [A Pass Number Instead of Clearing Marks](#a-pass-number-instead-of-clearing-marks)
    - [Types](#types)
    - [A Single Collection Pass](#a-single-collection-pass)
    - [Mark](#mark)
    - [Sweep](#sweep)
    - [Objects Allocated During Sweep](#objects-allocated-during-sweep)
    - [`DArray<T>` and Untyped Blocks](#darrayt-and-untyped-blocks)
    - [Inspection](#inspection)
  - [Telling a Managed Pointer from an Unmanaged One](#telling-a-managed-pointer-from-an-unmanaged-one)
  - [Limitations](#limitations)
  - [What Could Be Improved](#what-could-be-improved)
  - [Summary](#summary)

---

## Introduction

Memory management is one of the hardest topics in the project. There is no perfect solution here: every option has a price, and the only question is which price you are willing to pay.

The original task sounded modest: pass arguments and data between execution on the C++ side and execution of the script. The data must arrive intact, at least for some period of time. A whole subsystem grew out of that requirement, and this article describes how it changed and why it ended up the way it is now.

---

## Early Versions

### Values Instead of Pointers

At first, for simplicity, strings were passed by value rather than by pointer. Complex structures were not planned to be passed at all, only scalar values. But as the project grew, this had to be abandoned.

### Reference Counting

The first version of the memory manager was reference counting. Objects were created on the heap, and a reference to an object pointed at the data itself. A few bytes in front of the data held the manager's metadata: the reference counter and debugging fields.

The system seemed simple and efficient, but as the project grew and was tested, it turned out to have more problems than benefits. The virtual machine works with its own registers and, of course, manages the counters. But the same management would have had to be added to all the other code that handles these pointers in any way. More and more entities appeared whose job was to guarantee that the counter is incremented at the right moment and decremented at another. The complexity grew like an avalanche, and I abandoned this scheme.

### Per-Frame Data

I started over, and first I fixed a convention: data passed between the two execution systems is short-lived and does not persist across several frames. Under this rule, any data produced is simply deleted at the end of the frame. This is not garbage collection: all the containers created in all the pools are reset. The operation is practically free, since it changes only a few variables that track how full the pool is.

But during implementation it became clear that this was a half-measure. Long-lived data is needed: for example, a state machine in a script may create a structure and hold onto it for a while. Without that, the project cannot be completed.

### The Garbage Collector

I had a negative experience with garbage collectors in C# and Unity, so I spent a good deal of effort trying to do without one. But in the end, a garbage collector appeared in the working version anyway. It is not the most efficient way to manage memory, and the implementation is far from the best or the fastest. A simple implementation in the SOOT style suits me well, and I consider the collector a necessary evil that had to be written.

---

## How the Memory Manager Stays Independent of the VM

The virtual machine code sits at a higher level than the memory manager. The manager must not know about the VM: otherwise there would be a circular dependency, where the VM knows about memory, memory knows about the VM, and neither part can be built or tested separately.

To break this dependency, I built a type descriptor system. Inside the memory manager there is a table into which the virtual machine registers a short description of its types: what fields they have, where the pointers lie in the structure, and how to traverse them. The garbage collector reads this table and walks the structures, visiting all the pointers without knowing what these types are or where they came from. As a result, the memory manager does not depend on the VM, and the VM does not depend on a specific collector implementation.

---

## How the Garbage Collector Works

### Overall Design

SOOT uses a **non-moving mark-and-sweep collector**. Two features distinguish it from the textbook variant.

First, the collector does not look for roots itself. Roots are everything the VM keeps reachable: registers, global slots, the stack. During the mark phase the VM walks them and calls `mark_object` for each one. The collector relies on the VM to mark everything that is needed.

Second, the phases fit into a frame differently. Marking runs in one piece while scripts are not running (stop-the-world). Sweep can be spread over several frames: it runs with a budget and processes a few blocks per call.

An object marked in the current pass stays alive. An object that nobody marked during the pass is considered garbage and is freed. There is no object relocation and no compaction, so pointers to objects never change.

### Memory: the Region and Arenas

The whole heap is a single reserved region of virtual address space of up to `max_mb`. On Linux it is created with `mmap` and the `MAP_NORESERVE` flag, and physical pages are allocated on first access. On Windows `VirtualAlloc` is used: for now the region is reserved and immediately committed in full, which is a place for improvement.

Inside the region, a bump pointer carves out **arenas**, 1 MB each by default. Every object goes into one of nine **size classes**: 16, 32, 64, 128, 256, 512, 1024, 2048, and 4096 bytes. Each class has its own arena, and when it fills up, the next one is created.

Within an arena, all blocks have the same size. A block is handed out either from the freelist (freed blocks) or from the bump area (new blocks at the end of the arena). Each arena has an occupancy bitmap with one bit per block. Thanks to it, sweep walks the blocks directly, with no separate list of objects.

### The Object Header

In front of the data of every object sits a `GCHeader`: 8 bytes, or 16 if the debug name is enabled.

- `type_index` is the index of the type in `TypeRegistry`;
- `flags` holds the `MARKED` and `DEAD` flags, the pass number `pass_cycle` (1..15), and the size class number.

The header is taken into account when choosing a class. The block size is computed like this: the header plus the data, rounded up to 16 bytes, and then a class is chosen. The maximum object, including the header, cannot exceed 4096 bytes.

### A Pass Number Instead of Clearing Marks

The main feature of the scheme: "alive" means "marked in the current pass," not "has a raised flag." Each pass gets a number from 1 to 15 (0 is skipped), and `pass_cycle` in the header stores the number of the last pass in which the object was marked.

Because of this, there is no need to clear the marks of all objects before a pass begins. The new number itself invalidates all the old marks. Fifteen values are enough because every pass is driven through to the end of sweep, so the number only has to differ from the previous one.

### Types

`TypeRegistry` stores, for each type, the size, the alignment, a list of pointer fields (`PtrField`: `type_index`, `offset`, `array_count`), and two delegates:

- `mark` walks the pointers inside an object;
- `destroy` releases non-GC resources, for example a `std::string`. The block's memory itself is returned by sweep, so `destroy` should not call `gc::release`.

Index 0 is reserved as a sentinel: "untyped block." The collector does not scan such blocks.

### A Single Collection Pass

Schematically, the order is as follows:

```cpp
gc::start_pass();            // new pass number, sweep cursor reset
// the VM walks the roots and calls gc::mark_object(...) for each one
while (!gc::sweep_pass(64)) {
    // hand control back to the frame and continue next time
}
gc::finish_pass();
```

### Mark

`mark_object(payload)`:

- checks that the pointer lies in the heap and that the object is not marked as dead;
- if `pass_cycle` already equals the current one, the object is marked, and the traversal ends there;
- otherwise it writes the current number into the header, sets `MARKED`, and calls the type's `mark` delegate.

The delegate walks the pointers inside the object and calls the same `mark_object` for each of them through a small trampoline. This way nested structures are traversed by the same code, and cyclic references do not cause infinite loops: a repeated visit to an already marked object returns immediately.

### Sweep

`sweep_pass(budget)` walks the arenas and blocks with a budget: per call it processes a limited number of occupied blocks and returns `false` until the work is finished. For each occupied block:

- if `pass_cycle` equals the current one, the object is alive and `MARKED` is cleared from it;
- otherwise it is garbage: `destroy` is called (if there is one), the block is returned to the arena, and the `total_freed` counter grows.

The budget exists so that sweep does not block the game for long. The `collect()` function runs `sweep_pass` with a huge budget until it completes. It does not walk the roots, so it is suitable only where everything needed has already been marked or where freeing everything else is acceptable.

### Objects Allocated During Sweep

A new object gets the current pass number in its header. So a sweep that is running at that moment will not touch it: the object counts as alive in the current pass. This simplifies incremental sweep: scripts may allocate memory between calls, and the new objects will not disappear. In the next pass, such an object will be freed if nobody marks it.

This gives a rule for the mark phase: while it runs, scripts do not execute. A new object with the current number would count as already marked, and `mark_object` would not walk its pointers.

### `DArray<T>` and Untyped Blocks

`DArray<T>` is a dynamic array on the heap. Its buffer is allocated through `gc::alloc(/*sid=*/0, ...)`, that is, as an untyped block that the collector does not scan.

- For `DArray<i32>` and `DArray<f32>` this is enough: there are no pointers inside.
- For `DArray<T*>` this does not work: the pointers inside will not be marked, and the objects they point to will be collected at the next sweep. For arrays of pointers, `RuntimeArray` is used, with a mark delegate for each kind of element.

The buffer itself also follows the general rules. If nobody marks it in a pass, sweep will free it, so whoever owns the array must mark the buffer in its own `mark` delegate.

### Inspection

`for_each_live` walks all live objects on the heap and calls a visitor with the type, the pointer, and the size. The `gcls` inspector and debugging tools use it.

---

## Telling a Managed Pointer from an Unmanaged One

From time to time code needs to know what a pointer points to: an object managed by the garbage collector, or some unrelated data. In SOOT this is solved simply. All arenas lie in one contiguous region with a known start and end, and a small predicate function, `is_in_heap`, checks whether a pointer falls in that range. That is enough to answer the question.

There is one condition. The check only answers "the address is inside the region." For the header to be read correctly, the pointer must point exactly at the start of the object's data, not at one of its inner fields.

---

## Limitations

- **Maximum object size** is about 4 KB including the header. There are no larger objects on the heap, and a request for a larger size fails. For `DArray` this means that an array that does not fit into 4 KB currently cannot grow.
- **Recursive marking** uses the system stack. A very long chain of objects (for example, a linked list of many thousands of elements) can overflow it.
- **Arena lookup** when freeing a block is linear: it goes through all the arenas.
- **Allocation looks only at the last arena of a class.** Free blocks in earlier arenas are not reused until the last one fills up.
- **There is no compaction**, so fragmentation is possible.

---

## What Could Be Improved

**Specialized pools for vectors and quaternions.** These are the most frequent structures, and for them the header is currently expensive. A 16-byte vector plus an 8-byte header makes 24 bytes, but sizes are rounded to 16 and then to a class, so the vector occupies a block of class 32. The overhead is 100%. If such objects were stored in a separate pool where a block consists only of data and the type is known from the pool itself, a vector would take 16 bytes, half as much. The size-class lookup and the header write would disappear as well. This is not a rewrite of the collector, but the addition of one or two specialized pools next to the general one.

**An explicit marking stack** instead of recursion would remove the risk of overflowing the system stack.

**Large objects.** A separate path for blocks larger than 4 KB would remove the size limit, in particular for `DArray`.

**Fast arena lookup and allocation from any arena of a class** would reduce the cost of freeing and fragmentation. Arena lookup could be replaced by, for example, dividing the address by the arena size (arenas of equal size lie in the region one after another).

---

## Summary

The garbage collector in SOOT does not aim to be efficient. It is non-moving, with no compaction and no parallelism, and it has no generations in the classical sense, although the pass number in the header gives something similar to an age. But it is simple, predictable, and sufficient for the tasks the language faces: game code, where most data lives for a single frame, while long-lived structures are rare and few in number.

As in the other parts of the project, a simple option was chosen where a complex one is not justified. The limitations are known and written down, and the ways to overcome them are clear.

---

[Project Status](./project-status.md)
