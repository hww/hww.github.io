# SOOT: Scriptable Object-Oriented Toolkit

**An Embedded Lisp Environment for Runtime Orchestration, Build Automation, and Constrained Hardware**

[**GitHub Repository →**](https://github.com/hww/soot) · [**Script Language Reference →**](https://github.com/hww/soot/blob/main/common/script/README.md) · [**Standard Library Documentation →**](https://github.com/hww/soot/blob/main/common/script/README.LIB.md)

SOOT is an embeddable Lisp interpreter and automation environment designed to orchestrate development workflows for resource-constrained platforms, including CPUs, microcontrollers, and digital signal processors.

The project explores how a programmable, interactive environment can unify source transformation, build orchestration, target deployment, and runtime inspection. Instead of relying entirely on external scripts and disconnected tools, developers can use Lisp to describe and control the development process through a single extensible environment.

SOOT is part of a broader ecosystem that includes **Carbon**, a dedicated execution layer with its own virtual machine and compiler.

## Architecture: SOOT and Carbon

The ecosystem separates high-level development automation from low-level code execution.

### SOOT — Scripting and Orchestration

SOOT provides the high-level control layer. Its Lisp interpreter can be used to automate development tasks, transform source files, coordinate compilation and linking, and manage interactions with target hardware.

Key capabilities include:

- **Embedded Lisp interpreter:** A programmable scripting environment designed for integration with development tools and constrained systems.
- **Interactive REPL:** An iterative environment for evaluating expressions, experimenting with code, inspecting state, and debugging automation workflows.
- **Build orchestration:** Lisp scripts can coordinate build steps, invoke external tools, and manage project-specific workflows.
- **Source preprocessing:** Scripts can modify source files dynamically as part of a build or deployment pipeline.
- **Project-aware file operations:** Project context and configurable path resolution support reusable automation scripts.
- **Network REPL:** A network-accessible interactive interface supports remote development workflows and integration with external editors.

### Carbon — Virtual Machine and Compiler

Carbon is the execution layer associated with SOOT. It provides a dedicated virtual machine and a compiler intended to connect high-level development workflows with code execution on target platforms.

The **Carbon Virtual Machine is functional**, while the **Carbon Compiler remains under active development**. The compiler should therefore be considered an ongoing part of the project rather than a completed component.

Keeping the interpreter and execution layer separate allows SOOT to coordinate the development process without making its orchestration logic dependent on the implementation details of the target execution environment.

## Interactive Development Workflow

The REPL is central to SOOT's development model. It provides a feedback loop in which scripts and tools can be tested and refined interactively rather than only through repeated, fully automated builds.

A typical workflow can include:

1. **Prepare:** Configure a project and define its build or deployment procedures.
2. **Transform:** Use Lisp scripts to preprocess or generate source files.
3. **Orchestrate:** Coordinate compilation, linking, and other external tools.
4. **Deploy:** Automate