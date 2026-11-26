<div style="display: flex; align-items: flex-start; gap: 20px; margin-bottom: 25px; flex-wrap: wrap;">
  <div style="flex-shrink: 0;">
    <img src="images/wendy.png" alt="hww" width="120" style="border-radius: 8px; display: block; box-shadow: 0 2px 8px rgba(0,0,0,0.1);"/>
  </div>
  <div style="flex: 1; min-width: 280px; line-height: 1.6;">
    <h1 style="margin-top: 0; margin-bottom: 6px; border-bottom: none; padding-bottom: 0; font-size: 2em; font-weight: 700;">Valeriya Pudova (hww)</h1>
    <p style="margin: 0 0 10px 0; font-size: 1.1em; color: #24292e; font-weight: 600;">
      Systems Architect &nbsp;|&nbsp; R&D Engineer &nbsp;
    </p>
    <p>Hardware-software systems · low-level software · runtime architecture</p>
    <p style="margin: 0 0 14px 0; font-size: 0.95em;">
      🌐 <a href="https://github.com/hww" target="_blank" rel="noopener">GitHub Profile</a> &nbsp;|&nbsp; 
      💼 <a href="https://www.linkedin.com/in/valeriyap/" target="_blank" rel="noopener">LinkedIn</a> &nbsp;|&nbsp; 
      ✉️ Telegram: <a href="https://t.me/valery_h2w" target="_blank" rel="noopener">@valery_h2w</a> &nbsp;|&nbsp; 
      ✉️ Email: <a href="mailto:valery.hww@gmail.com">valery.hww@gmail.com</a>
    </p>
  </div>
</div>

------------------------------

## Professional Profile

I am a systems architect and R&D engineer working across system software, embedded development, and hardware engineering. I design and implement complex systems end to end — from electronic schematics and multilayer PCBs to embedded firmware, FPGA logic, interpreters, compilers, virtual machines, and runtime environments.

My engineering background is rooted in game-engine development and research into runtime architecture. At SyberSoft and Rocket Amusements, I held lead development roles and worked on game engines, software, and hardware platforms for arcade systems. My professional work has also involved collaboration with Ronald Siller and David Siller. I also conduct independent architectural research and low-level analysis of commercial AAA game-engine pipelines, including those of Naughty Dog and Insomniac Games.

In software engineering, I focus on execution architecture, deterministic systems, virtual machines, domain-specific languages (DSLs), embedded Lisp/Scheme environments, metaprogramming, and data-driven architectures. In hardware engineering, my work includes circuit design, PCB layout, signal and power integrity analysis (SI/PI), microcontrollers, FPGA, DSP, and prototyping.

I also operate an independent hardware development and prototyping lab equipped for electrical measurement, machining, CNC work, and additive manufacturing. This infrastructure supports the development of integrated hardware-software prototypes, from PCB and mechanical design to firmware and target runtime software.

## Skills & Competencies

### System Software & Architecture

- **Architecture and runtimes:** game engines, deterministic runtime systems and virtual machines, interpreters, compilers, and translators.
- **Programming languages:** C/C++, C#, Go, Lisp, Scheme (Gambit-C), Python, Ruby.
- **Graphics and mathematics:** 3D mathematics, shader development, graphics pipelines.
- **Performance engineering:** low-level optimization for performance, code size, and scalability; architecture designed for efficiency and maintainability.
- **Tools and environments:** Unity, Gamebryo, UDK/Unreal Engine, Qt Creator, JetBrains Rider, MSVC, Emacs.

### Embedded Systems & Hardware Engineering

- **Circuit design and PCB layout:** digital and analog circuit design; multilayer PCB layout, including boards with four or more layers (Altium Designer, Mentor Graphics Xpedition, PADS, KiCad).
- **Signal and power integrity:** SI/PI analysis and noise reduction in measurement circuits using HyperLynx.
- **Embedded systems and firmware:** microcontroller software for bare-metal environments and RTOS-based platforms, including Zephyr; ESP-IDF and STM32CubeIDE; Nordic, ESP32, STM32, and RP2040 MCUs.
- **FPGA:** RTL development and verification using Verilog/SystemVerilog, Verilator, Lattice, Altera/Intel, and Xilinx tools.
- **Mechanical design and enclosures:** components and enclosures for industrial production and 3D printing (Siemens NX, SolidWorks, PTC Creo).

## Academic R&D & Courses

| Institution / Platform | Year | Course / Program | Role / Result |
|---|---:|---|---|
| BerkeleyX (edX) | 2014 | CS184.1x: Foundations of Computer Graphics | Teaching Assistant |
| BerkeleyX (edX) | 2013 | CS184.1x: Foundations of Computer Graphics | Teaching Assistant |
| BerkeleyX (edX) | 2012 | CS184.1x: Foundations of Computer Graphics | Final score: 100% |
| BerkeleyX (edX) | 2012 | CS188.1x: Artificial Intelligence | Final score: 97% |

**Languages:**

- Russian — Native.
- English — Fluent / Professional Proficiency.

## Work Experience & R&D Practice

### R&D Lab — Independent Engineering Practice

**Founder, Systems Architect & R&D Engineer** · 2023–Present · Pskov, Russia · Remote / Contract Work

#### Arcade Machine Hardware-Software System

- **Runtime architecture (C# / Unity):** designed and implemented a deterministic runtime for game logic. Developed a hierarchical process system, a data-oriented ECS architecture, and a custom task manager to control the game loop and coordinate task execution while reducing the risk of data races.
- **Technical leadership:** led development and designed the architecture for Rocket Amusements projects, including Crazy Claw — Rings and Monster Blaster.
- **CARBON virtual machine:** developed an isolated, cross-platform execution environment for low-level code, separating application logic from the target hardware platform.
- **SOOT Lisp interpreter:** developed an interpreter for metaprogramming, cross-compilation, and dynamic resource management. Designed a multi-pass cross-compiler for 8-bit architectures with automatic memory-offset calculation.
- **Network stack and backend (Go):** developed a UDP server for real-time communication, a custom binary data-exchange protocol, an HTTP monitoring server, and an SQLite storage layer.
- **Hardware and firmware:** designed the architecture and PCB layout of several multilayer I/O boards based on Nordic MCUs and ESP32. Developed C++ firmware for peripheral signal handling on bare-metal and RTOS-based platforms, including Zephyr.

#### Wearable Optical Monitoring and 3D Telemetry Visualization System (PPG)

- **Analog front end (AFE):** designed the schematics and PCB layout for sensor boards in Altium Designer; performed signal and power integrity (SI/PI) analysis to reduce noise in the measurement path.
- **Firmware (C++ / ESP32):** developed firmware for real-time digitization, filtering, and initial mathematical processing of pulse signals, with telemetry transmitted over UDP.

#### SODIMM DSP Module

- Designed the schematic and multilayer PCB layout for a DSP module in the SODIMM form factor, based on an Analog Devices DSP processor and DDR3 memory. Performed simulation and optimization in HyperLynx.

### VedaProject

**Lead Software Engineer / Development Team Lead** · 2023 · Moscow, Russia

Project: a trusted software environment for critical infrastructure, with protection against insider attacks and secure updates through an air-gapped architecture.

- **Software development and information security:** designed the trusted-environment system. Implemented full-disk and per-file OS encryption, as well as a public key infrastructure (PKI) based on OpenSSL and CryptoAPI.
- **Compilation and GUI:** created a client-server distributed compilation system for trusted software on Debian Linux and developed Qt-based monitoring applications.
- **Management:** led an engineering team and conducted technical interviews.
- **Patent practice:** prepared the documentation and obtained Rospatent registration certificates for software systems No. 2023664550 and No. 2023663992.
- **Technologies:** C++, Qt, Ruby on Rails, Python, Racket, Dart/Flutter, Bash, Linux.

### Arcadia VR

**CTO / Head of Hardware & Software** · 2020–2023 · Moscow, Russia · Remote

- **Technical and strategic leadership:** defined the company’s technical direction, audited the codebase, conducted code reviews, and provided technical and business consulting. Helped build the engineering team and mentored developers.
- Deployed internal server infrastructure and automated CI/CD pipelines and build farms, supporting releases of REQUISITION VR and Jentrix.
- **Device R&D:**
  - designed and built several prototypes of inertial motion-tracking gloves;
  - designed a haptic-feedback VR rifle; modelled a custom linear motor for an MMA actuator in COMSOL Multiphysics and built several prototypes;
  - designed enclosures for FDM 3D printing in Siemens NX.
- **Technologies:** C#, Unity, C++, Python, RP2040, ESP32 and STM32 MCUs, Protobuf, UDP, Extended Kalman Filter (EKF), Madgwick and Mahony filters for IMU/AHRS processing.

### Rocket Amusements

**Lead Software & Hardware Engineer** · 2014–2020 · Saint Petersburg, Russia / New Braunfels, USA / Guangzhou, China

- Worked on collaborative projects with game designer and producer Dave Siller.
- **Electronics design:** designed two revisions of programmable I/O boards for arcade machines.
- **Mechanical and enclosure design:** designed the mechanical structure and performed strength calculations for a two-seat ticket-redemption arcade cabinet in SolidWorks and PTC Creo, preparing it for serial production. Designed plastic components for injection molding.
- **3D game development:** developed Unity software for a range of 3D arcade games released into production in China:
  - Crazy Claw Original — local multiplayer for two players;
  - Crazy Claw Junior — a single-player version for children;
  - Crazy Claw the Emojis — a competitive mode for three players on one screen.

### DigitalChile

**Lead Software Developer** · 2008–2014 · Saint Petersburg, Russia / New Braunfels, USA

- **Scripting runtimes:** developed and implemented a low-level integration (binding) of the Gambit-C Scheme/Lisp compiler with the commercial Gamebryo 3D engine for metaprogramming and data-driven gameplay logic.
- **Development tools:** created a custom game Lisp framework for Unity and released a game prototype built on it to validate the architecture.
- **Enterprise systems:** developed a reliable web-based automation and learning management system (LMS) for commercial and passenger vehicle insurance. Created an automation engine for 48 interactive video courses with branching logic.

### GameTools

**Lead Hardware-Software Solutions Engineer** · 2004–2008 · New Braunfels, USA / Saint Petersburg, Russia

- **Nintendo GBA/DS development:** designed the hardware architecture and implementation of the ViewBoy device for Nintendo Game Boy Advance SP and Nintendo DS handheld consoles.
- **Enclosure design and specifications:** designed the plastic enclosure for a cheat device for the Nintendo Famicom (the Japanese version of the NES). The design met the required dimensions and PCB edge-connector positioning specifications.

### SyberSoft

**Technical Director & Lead Developer** · 2002–2004 · Riga, Latvia / Limassol, Cyprus

- Led the game development team.
- The team developed SyberEngine, an in-house 3D engine designed for rapid prototyping and game production. It was used to build prototypes of the 3D shooter Metal Dogs and the arcade racing game Psycho Racers.
- **JAMMA platforms:** developed two generations of arcade interface boards connecting arcade cabinets to PC-compatible motherboards. The boards included VGA-to-TV conversion, USB HID support, USB DAC audio, and USB mass storage.
- Contributed to technical prototyping and hardware development for collaborative projects with Dave Siller, including The Secret Life of Guido, Metal Dogs, and Psycho Racers.

### REMIS Lab

**Lead Software & Electronics Developer** · 1998–2002 · Omsk, Russia

- **Hardware sound synthesis (DSP):** designed two versions of a digital sampler synthesizer based on Altera FPGA logic and a Motorola fixed-point DSP.
- **DSP firmware (C):** developed the synthesizer software in C, including a MIDI player. Created a CLI tool for building and compiling sound-bank data.
- **Console emulation:** developed the PVBackup hardware ROM emulator for Nintendo consoles, a custom Nintendo 64 cartridge, and a demonstration mini-game for the N64.

### PATISONIC

**Lead Systems Architect / PC Developer** · 1990–1997 · Omsk, Russia

- **Aleste 520EX computer:** designed the architecture and schematics of the Aleste 520EX personal computer. It provided backward compatibility with the Amstrad CPC 6128, enhanced graphics capabilities, and ran MSX-DOS.
- **MagicSound audio card:** designed the MagicSound audio expansion board for the Aleste 520EX and developed its software package.

### Aviation Technical College

**Student Engineer** · 1984–1988 · Omsk, Russia

- As part of my diploma project and research practice, designed and assembled the “Raduga” microcomputer, brought up its hardware, and developed its low-level system monitor software.

