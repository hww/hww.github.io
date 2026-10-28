# Anatomy of the SOOT Interpreter: Three Engineering Decisions

## Table of Contents

- [Introduction](#introduction)
- [Where the Interpreter Came From](#where-the-interpreter-came-from)
- [How Not to Lose the Source Code on the Way to Execution](#how-not-to-lose-the-source-code-on-the-way-to-execution)
- [Declarative Argument Checking](#declarative-argument-checking)
- [Error Handling: From a Message to Control of Execution](#error-handling-from-a-message-to-control-of-execution)
- [How These Decisions Complement Each Other](#how-these-decisions-complement-each-other)
- [Conclusion](#conclusion)

---

## Introduction

An interpreter for a Lisp dialect, even a small one, solves many tasks: it reads source text, turns it into expressions, evaluates them, checks function arguments, and reports errors. Each of these tasks is well studied on its own. The harder part is organizing their interaction so that the code stays understandable and the interpreter's behavior stays predictable.

This article looks at three decisions from the SOOT interpreter:

- storing information about the source code separately from the S-expressions themselves, while still being able to link the two (`TextDb`);
- a declarative description of the argument contract of built-in functions (`vararg_check`);
- error handling with trace accumulation and user-defined handlers written in the language itself.

They have one thing in common: each spares the code from repeating complex infrastructure logic in many places.

---

## Where the Interpreter Came From

> The SOOT interpreter grew out of GOOS, the scripting Lisp of the OpenGOAL project, which is distributed as an open-source project. The ideas behind `TextDb` and `vararg_check`, as well as part of the evaluator's structure, go back to it. This article describes how these mechanisms work in SOOT, including what has been changed in them. The names of projects belong to their respective rights holders, and SOOT is not affiliated with them.

---

## How Not to Lose the Source Code on the Way to Execution

### An Expression Is Not All the Information About a Program

Suppose the user wrote:

```lisp
(+ 10 (* 2 3))
```

The Reader, the component that reads text and recognizes expressions, turns this record into a tree of S-expressions: symbols, numbers, and lists. After that, evaluation hardly needs the source text. The interpreter only has to know that it must call `+` with the number `10` and the result of `(* 2 3)`.

But what if the user writes `(+ 10 (* 2 "abc"))`? It is not enough to report that the operation failed. The interpreter must show **where exactly in the source text** the problem occurred; otherwise, hunting for an error in a large file turns into torture.

The difficulty is that after reading, the interpreter works with language objects. How can these objects be linked to the places in the text they came from?

### Two Ways to Store Positions

The first way is to put the source information right into the tree nodes. Each node carries a file, a line, and a position. This is reasonable, but it ties the representation of an expression to diagnostics: the data has to be carefully carried through all levels of representation.

The second way, used in SOOT, is a separate store called **`TextDb`**. It maps language objects to information about their origin, while the objects themselves know nothing about it. The key is the identity of the object.

The scheme looks like this:

```txt
Source text
      |
      v
    Reader ------- link(pair, source, offset) -------+
      |                                              |
      v                                              v
S-expressions                                     TextDb
      |                                              |
      v                                              |
 Evaluation                                          |
      |                                              |
      v                                              |
Error in form F  ---- get_info_for(F) ---------------+
                                  |
                                  v
                    file, line, line text, and the ^ pointer
```

`TextDb` does not take part in evaluation. Its only task is to answer the question "where did this object come from?" for a given object.

### How the Store Is Organized

`TextDb` holds two structures:

- a list of text sources (`SourceText`): a file, a program string, or REPL input;
- a table that maps a heap object to a `TextRef` record. The record consists of a reference to the source and an offset into it.

When the Reader creates a pair, it immediately calls `link(pair, source, offset)`. `SourceText` has a table of line-start offsets built in advance, so given an offset it is easy to find the line number, the line text, and the column.

Two kinds of reports are built from the position table. The detailed one, `get_info_for`, prints the file and line number, the line itself, and a red `^` pointer under the right character. The short one, `get_short_info_for`, returns a `ShortInfo` structure (file, line, column, line text) that is convenient for editors and the REPL. If an offset falls outside the text, it is clamped to the end of the text so that diagnostics do not fail themselves.

### Only Pairs Have Positions

There is an important detail. Only a heap object can be a key, so `link` accepts only pairs. Numbers, symbols (`symbol`), and individual character literals (`char`) are stored directly inside `Object` and have no identity of their own. Besides, symbols are interned: the same symbol appears in dozens of places and cannot have a single position.

The practical consequence is that an error points to a form, that is, to the list cell in which it occurred, rather than to a specific number or symbol inside it. For the call `(string-length 42)`, the diagnostics will show the line with this call and an arrow at the start of the form.

### What Happens to Positions During Transformations

Code is not always written by hand. Quasiquote builds new lists from a template, and the new pairs have no position of their own in the source. For this, `TextDb` has transfer operations.

- `copy_link(from, to)` copies a position from one pair to another. Quasiquote uses it like this: each new cell receives the position of the corresponding template cell. So a form assembled from a macro template points to the place of that template. With `unquote-splicing`, only the first inserted element receives a position, and the rest follow without a link of their own.
- `inherit_info(parent, child)` walks all pairs inside `child` and assigns them the position of `parent`. Pairs that are already linked are not overwritten: if a fragment came from the source, it keeps its true position. The walk does not descend into already known nodes.
- `clear_info()` resets the whole store, for example between REPL sessions.

### The Price of the Decision

The table key is a `shared_ptr` to the object, so `TextDb` keeps all linked pairs alive until `clear_info()` is called. For files and REPL sessions this is acceptable: memory is released on cleanup. For a long-running process that constantly generates code, it would be worth taking into account.

In short, `TextDb` separates diagnostic data from the representation of an expression and keeps the ability to go from an object back to a place in the text. It is not the only way, but it moves the handling of sources and coordinates into a separate subsystem.

---

## Declarative Argument Checking

### Why Built-in Function Arguments Are a Separate Task

A call in Lisp looks like a list: the first element is the operation, the rest are arguments. But functions expect different things. One needs exactly one argument of a particular type, another needs two, a third allows additional parameters with keys.

What should happen on `(string-length 42)`? Or on `(string-length)` or `(string-length "a" "b")`? Every built-in function must handle such cases correctly. If the checks are written by hand, the code grows overgrown with repetitive conditions: check the number of arguments, the type of the first, the type of the second, produce a message. And the more built-in functions there are, the more places there are where these rules have to be maintained separately.

### A Contract in One Line

In SOOT every built-in function starts with a call to `vararg_check`. It takes the call form (for the error message), the already prepared arguments, and two descriptions: the expected positional arguments and the expected named ones.

Here are real examples:

{% raw %}
```cpp
// (string-length s): one string
vararg_check(form, args, {{ObjectType::STRING}}, {});

// (string-ref s i): a string and an integer index
vararg_check(form, args, {{ObjectType::STRING}, {ObjectType::INT}}, {});

// (floor x): a number of either kind
vararg_check(form, args, {{ObjectType::INT, ObjectType::FLOAT}}, {});

// (define name value [:env environment])
vararg_check(form, args, {{ObjectType::SYMBOL}, {}}, {{"env", {false, {}}}});
```
{% endraw %}

Rules for reading such a description:

- the outer list describes argument positions, and the inner list enumerates the allowed types for that position;
- an empty inner list means "any type";
- if the outer list is non-empty, the number of positional arguments must match it exactly;
- named arguments are described by a dictionary: the name, a required flag, and the allowed types;
- a named argument not listed in the description is an error.

After that, the function can concentrate on its own work without repeating the contract check.

### Uniform Messages

All violations go through the same error-building functions, so the messages for all built-in functions look alike:

- wrong number of arguments: `arity mismatch: expected N arguments, but got M`;
- wrong type: `type error at argument [i]: expected one of [...], but got [...]` (the index is counted from zero);
- a missing, unexpected, or wrongly typed named argument: separate messages that include the key name.

Every message is followed by the list of passed arguments, and the error is attached to the call form, so `TextDb` can show the place in the source. Uniform behavior matters here no less than shorter code: the user gets consistent messages, and the author of the interpreter is spared dozens of similar fragments.

### Two Mechanisms for Two Tasks

`vararg_check` checks **types** inside a built-in function. Splitting the call into positional and named arguments (`:key value`) and evaluating the arguments are done by `get_args` and `eval_args` before the function is entered. For user functions and macros, the contract is described by a different structure, `ArgumentSpec`, which is built by parsing a parameter list such as `(x y &key z &rest body)`. So built-in functions and user code have two different but compatible ways of describing arguments.

### What the Mechanism Cannot Do

- An empty description of positional arguments means "do not check the number of arguments," so "exactly zero arguments" cannot be expressed this way.
- It is not possible to specify a range ("from one to three") or an optional positional argument. For such cases, a manual check is used.

This is deliberate moderation. Typical contracts are served by a common mechanism, and non-standard ones stay non-standard. Trying to make one mechanism describe every case would lead to a system more complex than the checks themselves.

---

## Error Handling: From a Message to Control of Execution

### An Error Must Not Lose Its Context

The user calls function A, it calls function B, and B finds an error. If the interpreter simply stops and prints a message, it will be hard to understand how the program arrived at the problematic operation. So an error must carry its context with it: where it occurred and through which calls it was reached.

Besides, an error is a matter of execution control. Deep inside an evaluation, the current operations have to be stopped correctly, the active calls have to be walked back through, and a decision has to be made whether someone should catch the error.

### How It Is Done

The evaluator is recursive: each form is evaluated by a call to `eval_with_rewind`, which calls `eval`. An error is thrown as a C++ exception, `EvalException`, and "stack unwinding" is the ordinary propagation of that exception through the nested evaluator calls.

At each level, `eval_with_rewind` catches the exception and adds a trace entry to it: the form being evaluated and the name of the environment. Then the exception either goes further up or is passed to a handler.

When the trace reaches the top, the report collects three things:

1. the error message;
2. the place in the source for the form where the error occurred (`TextDb`);
3. the list of accumulated entries, with a file and a line for each form that has them.

All three decisions work together here: `vararg_check` produces the message, `TextDb` determines the place, and stack unwinding accumulates the context.

A note on terminology. The call environment of a function (`EnvironmentObject`) stores the call form and the function itself. This is not the same as the virtual machine's `StackFrame`, which is described in another article of the series.

### Error Handlers in the Language Itself

The user can catch an error with the special form `with-error-handler`:

```lisp
(with-error-handler
  (lambda (message form trace) ...)
  ...body...)
```

The form evaluates the handler lambda and executes the body in a new environment to which the handler is attached. If an error occurs in the body, at any call depth, the exception reaches the level of the body, `eval_with_rewind` sees the attached handler and calls it with three arguments: the message, the form in which the error occurred, and the list of the accumulated trace.

What the handler returns determines the fate of the error:

| Handler result | What happens |
| --- | --- |
| empty list (`null`) | the error is suppressed, evaluation continues, and the form returns `null` |
| `#t` | the error is rethrown upward unchanged |
| a pair `(context message)` | one more entry is added to the trace, and the error goes upward |
| any other value | the error is suppressed, and the body form returns this value |

Because errors can be rethrown, handlers can be nested: an inner one can append context and pass the error to an outer one.

Keep in mind that by the time the handler is called, the stack has already been unwound to the body form. So the handler cannot resume evaluation from the point of the error. It can only replace the result of the form inside which the error occurred, or pass the error up. Catching an error does not guarantee that the program's state after it is fit to continue. It is a tool for control, not a guarantee of recovery.

### Good to Know

The evaluator is recursive and does not optimize tail calls, so the recursion depth of a script is limited by the size of the C++ stack. Errors are implemented with exceptions: on the normal path this costs almost nothing, but throwing an exception is more expensive than an ordinary return.

---

## How These Decisions Complement Each Other

The three mechanisms answer the three questions a person asks when an error occurs.

- **Where did the problem occur?** The link between S-expressions and the source (`TextDb`) gives the file, the line, and the place in the line.
- **What exactly is wrong with the call?** The argument contract (`vararg_check`) identifies typical violations and produces the message.
- **What happens next?** Error handling decides how the error propagates, what context accumulates, and when control passes to a user-defined handler.

Each decision is local and has a clear task, but together they make diagnostics coherent.

---

## Conclusion

When you write a small interpreter, attention goes to the language itself: numbers, strings, lists, functions, evaluation forms. As a project grows, the mechanisms around evaluation play an ever larger role.

You need to keep the link between expressions and the source so that errors are easy to find. You need to check arguments so that built-in functions behave uniformly. You need to organize error propagation so that the interpreter can not only report a problem but also hand control to a user-defined handler.

`TextDb` separates information about the source from the representation of an expression and keeps the link between them. `vararg_check` expresses a typical contract in one line instead of repetitive checking logic. The error system uses the execution context and lets the program take part in reacting to an error. All three decisions are fairly local, and none of them tries to turn the interpreter into a universal framework. They remove specific sources of complexity where that is justified, and in this the approach resembles the rest of SOOT's architecture: a simple solution is chosen wherever a complex one does not pay off.

---

[Project Status](./project-status.md)
