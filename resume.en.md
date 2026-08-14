# VALERIA PUDOVA

SYSTEM ARCHITECT & R&D ENGINEER

Design of hardware-software complexes (HSC), low-level software, and system runtimes

<div style="display: flex; align-items: flex-start; gap: 20px; margin-bottom: 25px; flex-wrap: wrap;">
  <div style="flex-shrink: 0;">
    <img src="pp/wendy.png" alt="hww" width="120" style="border-radius: 8px; display: block; box-shadow: 0 2px 8px rgba(0,0,0,0.1);"/>
  </div>
  <div style="flex: 1; min-width: 280px; line-height: 1.6;">
    <h1 style="margin-top: 0; margin-bottom: 6px; border-bottom: none; padding-bottom: 0; font-size: 2em; font-weight: 700;">Valeriya Pudova (hww)</h1>
    <p style="margin: 0 0 10px 0; font-size: 1.1em; color: #24292e; font-weight: 600;">
      Systems Architect &nbsp;|&nbsp; R&D Infrastructure Engineer &nbsp;|&nbsp; Low-Level Specialist
    </p>
    <p style="margin: 0 0 14px 0; font-size: 0.95em;">
      🌐 <a href="https://github.com">GitHub Profile</a> &nbsp;|&nbsp; 
      💼 <a href="https://linkedin.com">LinkedIn</a> &nbsp;|&nbsp; 
      ✉️ Telegram: <a href="https://t.me">@core_systems_eng</a>
    </p>
  </div>
</div>

------------------------------

## Professional Profile (Summary)

I am a full-cycle system architect and R&D engineer, specializing at the intersection of hardware development, low-level execution layers (runtimes), and high-performance software environments. My expertise covers the entire lifecycle of creating complex systems: from designing physical hardware (schematic design, multi-layer PCB routing, pre/post-layout signal integrity analysis, in-circuit programming, FPGA/HDL, and microcontrollers) to writing custom system software (language interpreters, virtual machines, compilers, and digital signal processing (DSP) for real-time sound synthesis).

The foundation of my engineering practice is rooted in game engine development and deep research into runtime architecture. I specialize in reverse engineering and low-level analysis of proprietary AAA engines of world-class caliber (including detailed breakdown of Naughty Dog and Insomniac Games pipelines). In software engineering, I focus on designing deterministic virtual machines, developing custom domain-specific languages (DSLs), and implementing high-speed scripting engines (such as embedded Lisp/Scheme-based runtimes) that replace heavy native code with flexible, data-driven pipelines.

Beyond software design, I run my own fully equipped hardware development and prototyping laboratory. The space combines professional electrical measurement and testing equipment with industrial machinery, including CNC systems, multi-axis milling, lathe work, and additive manufacturing (3D printing). This infrastructure allows me to independently and rapidly create complete hardware-software complexes (HSC) — from PCB routing and mechanical fabrication to writing custom runtimes for the target hardware.

------------------------------

## Skills and Competencies

### System Software and Game Engines

* Architecture and Runtimes: Design of game engines, custom deterministic virtual machines, interpreters, and translators.
* Programming Languages: Expert proficiency in C/C++, C#, Golang, Lisp, Scheme (Gambit-C), Python, Ruby.
* Graphics and Mathematics: Strong 3D math, shader development, graphics processing pipelines.
* Optimization: Low-level code optimization for performance, size, scalability, and architectural cleanliness.
* Tools and Environments: Unity 3D, Gamebryo, UDK (UE), Qt Creator, Rider, MSVC, Emacs.

### Hardware Engineering

* Schematic Design and PCB Layout: Design of digital and analog circuits, routing of multi-layer (up to 4+ layers) printed circuit boards (Altium Designer, Mentor Graphics Xpedition, PADS, KiCad).
* Signal Integrity Analysis: Professional SI/PI analysis and noise minimization in measurement circuits (HyperLynx).
* Embedded Systems (Firmware): Firmware development for microcontrollers on bare metal and on Zephyr RTOS, ESP-IDF, STM32CubeIDE (Nordic MCU, ESP32, STM32, RP2040).
* FPGA Design: Implementation and verification of FPGA logic (Verilog / SystemVerilog, Verilator, Lattice, Altera/Intel, Xilinx).
* Mechanics and Enclosures: Design of enclosures and components for industrial production and 3D printing (Siemens NX, SolidWorks, PTC Creo).

------------------------------

## Academic R&D and Courses

| Institution / Platform | Year | Course / Program | Role / Result |
| --- | --- | --- | --- |
| BerkeleyX (edX) | 2014 | CS184.1x: Foundations of Computer Graphics | Teaching Assistant |
| BerkeleyX (edX) | 2013 | CS184.1x: Foundations of Computer Graphics | Teaching Assistant |
| BerkeleyX (edX) | 2012 | CS184.1x: Foundations of Computer Graphics | Final Score: 100% |
| BerkeleyX (edX) | 2012 | CS188.1x: Artificial Intelligence | Final Score: 97% |

Languages:

* Russian: Native
* English: Fluent / Professional Proficiency

------------------------------

## Work Experience & R&D Practice

### R&D Lab (Sole Proprietorship)

Founder, System Architect & R&D Engineer
2023 — Present | Pskov, Russia (Remote / Contract Work)

#### Hardware-Software Complex (HSC) for an Arcade Machine from Scratch

* Runtime Architecture (C# / Unity): Designed and implemented a deterministic execution core (engine) for game logic. Developed a hierarchical process system, ECS architecture (Data-Oriented), and a custom Task Manager for strict control of the game loop and elimination of data races.
* CARBON Virtual Machine: Created an isolated cross-platform low-level code execution environment to abstract logic from the target hardware.
* SOOT LISP Interpreter: Developed a high-performance interpreter for metaprogramming, cross-compilation, and dynamic resource management. Designed a multi-pass cross-compiler for 8-bit architectures with automatic memory offset calculation.
* Network Stack and Backend (Golang): Wrote a lightweight UDP server for real-time communication, a custom binary data exchange protocol, an HTTP monitoring server, and an SQLite storage layer.
* Hardware: Designed the architecture and topology of several multi-layer I/O boards based on Nordic MCUs and ESP32. Wrote firmware in C++ (on bare metal / Zephyr / RTOS) for handling peripheral signals of the arcade machine.

#### Wearable R&D System for Optical Monitoring and 3D Telemetry Visualization (PPG)

* Analog Front-End (AFE): Designed and routed sensor PCBs in Altium Designer, performed signal and power integrity (SI/PI) analysis to minimize noise in the measurement path.
* Firmware (C++ / ESP32): Wrote firmware for digitization, filtering, and primary mathematical processing of pulse signals in real time, with telemetry transmission over UDP.

#### SODIMM DSP Module for Digital Signal Processing

* Designed the schematic and topology of a complex multi-layer DSP module PCB in SODIMM form factor based on the Analog Devices ADSP-21SC584 processor and DDR3 memory (with simulation and optimization performed in HyperLynx).

### VedaProject

Lead Software Engineer, Head of Development Group
2023 — 2023 | Moscow, Russia

Project: Trusted environment system, insider attack protection, and secure updating via a physically isolated (air-gapped) circuit for critical infrastructure.

* Development and InfoSec: Designed an end-to-end secure environment system. Implemented full-disk and per-file OS encryption mechanisms, and a public key infrastructure (PKI) based on OpenSSL / Crypto API, ensuring data integrity.
* Compilation and GUI: Created a client-server distributed compilation system for trusted software on Debian Linux. Developed Qt-based monitoring GUI applications.
* Management: Led a team of engineers, conducted technical interviews.
* Patent Practice: Prepared documentation and obtained Rospatent certificates for software complexes (No. 2023664550, No. 2023663992).
* Technologies: C++, Qt, Ruby (RoR), Python, Racket, Dart/Flutter, Bash, Linux.

### Arcadia VR

CTO / Head of Hardware & Software Department
2020 — 2023 | Moscow, Russia (Remote)

* Strategic Management: Defined the company's technical direction, conducted codebase audits, code reviews, and business consulting. Built a hiring pipeline, trained and mentored a team of engineers.
* Deployed internal server network and automated CI/CD build farms, ensuring releases of flagship VR hits (REQUISITION VR, Jentrix).
* Device Development (R&D):
  * Designed and produced several prototypes of motion capture gloves with inertial tracking.
  * Designed a VR rifle with haptic feedback. Modeled a custom linear motor for an MMA actuator in COMSOL Multiphysics. Manufactured several prototypes.
  * Mechanics and Infrastructure: Designed a series of enclosures for FDM 3D printing in Siemens NX.

* Technologies: C#, Unity, C++, Python, MCUs RP2040, ESP32, STM32, Protobuf, UDP, Kalman (EKF)/Madgwick/Mahony filters for IMU/AHRS.

### Rocket Amusements

Lead Software & Hardware Developer
2014 — 2020 | Saint Petersburg (RF) / New Braunfels (USA) / Guangzhou (China)

* Electronics Design: Designed from scratch two revisions of programmable I/O boards for arcade machines.
* Enclosure Design: Performed mechanical design and strength calculations for a two-seat arcade cabinet (ticket redemption class) in SolidWorks and PTC Creo for serial production. Designed injection-molded plastic components.
* 3D Game Development: Wrote game software in Unity for a line of 3D arcade games, put into mass production in China:
  * Crazy Claw Original (Local Multiplayer for two players).
  * Crazy Claw Junior (single-player children's version).
  * Crazy Claw the Emojis (competitive mode for 3 players on one screen).

### DigitalChile

Lead Software Developer
2008 — 2014 | Saint Petersburg (RF) / New Braunfels (USA)

* Scripting Runtimes: Developed and implemented a complete low-level integration (binding) of the Gambit-C Scheme/LISP compiler with the commercial 3D engine Gamebryo for metaprogramming and data-driven logic control.
* Development Tooling: Created a custom game LISP framework for integration with the Unity 3D environment and released a game prototype on it for architecture validation.
* Enterprise Systems: Developed a high-reliability web-based automation and learning management system (LMS) for corporate driver insurance (commercial and passenger vehicles). Created an automation engine for 48 interactive video courses with branched logic.

### GameTools

Lead Hardware-Software Solutions Engineer
2004 — 2008 | New Braunfels (USA) / Saint Petersburg (RF)

* Nintendo GBA/DS Design: Implemented the hardware architecture and design of the ViewBoy device for Nintendo GameBoy Advance SP and Nintendo DS portable consoles.
* Enclosures and Specifications: Designed the plastic casing for the Nintendo Famicom (Japanese NES) cheat device. The housing design strictly adhered to Nintendo's stringent industrial specifications for dimensions and PCB edge connector positioning.

### Sybersoft

Technical Director & Lead Developer
2002 — 2004 | Riga (Latvia) / Limassol (Cyprus)

* Led a team of developers in the game department.
* The team developed the internal 3D game engine SyberEngine, optimized for rapid prototyping and game production. Game prototypes created on it included the 3D shooter "Metal Dogs" and the arcade racing comedy "Psycho Racers".
* JAMMA Platform Development: Developed two generations of JAMMA-standard arcade platforms for interfacing arcade cabinets with PC-compatible motherboards. The boards included an advanced VGA2TV video converter, USB-HID support, USB-DAC audio systems, and USB bulk storage.

### REMIS Lab

Lead Software and Electronics Developer
1998 — 2002 | Omsk, Russia

* Hardware Sound Synthesis (DSP): Designed two versions of a digital sampler synthesizer based on Altera FPGA programmable logic and a Motorola DSP fixed-point signal processor.
* DSP Firmware (C): Wrote firmware for the synthesizer — a full-featured sampler synthesizer with a MIDI player in C. Created a CLI tool for building and compiling sound bank memory.
* Console Emulation: Developed the PVBackup hardware ROM emulator for Nintendo consoles, as well as a custom Nintendo 64 cartridge and a demonstration mini-game for the N64.

### PATISONIC

Lead System Architect / PC Developer
1990 — 1997 | Omsk, Russia

* Aleste 520EX Computer Development: Designed the architecture and schematics of the Aleste 520EX personal computer. The computer featured backward compatibility with the Amstrad CPC 6128, enhanced graphics capabilities, and ran the MSX DOS operating system.
* MagicSound Sound Card: Designed the custom MagicSound expansion sound card for the Aleste 520EX computer. Personally wrote the sound software package for this platform.

### Aviation Technical College

Student Engineer
1984 — 1988 | Omsk, Russia

* "Raduga" Microcomputer: As part of my thesis / research practice, fully designed, assembled, and booted the hardware of the "Raduga" microcomputer, and wrote its low-level system monitor software.
