# Embedded Systems, Hardware & Integrated Systems

Engineering projects covering hardware architecture, electronics, FPGA, embedded firmware, DSP, and integrated hardware-software products.

Projects are grouped by the scope and nature of my engineering contribution rather than by individual technologies. Each project appears in one category only. Additional technologies and implementation aspects are indicated through topic tags.

The categories distinguish between hardware design, hardware development with embedded firmware, complete end-to-end systems, and software projects related to hardware platforms.

---

## 1. Hardware Design & Digital Logic

Projects primarily focused on hardware architecture, digital logic, RTL design, electronic schematics, PCB development, and hardware interfaces.

Some projects also include software, emulators, or firmware developed by other team members or as supporting tools.

- **[LX-GPU — Programmable 2D Graphics Coprocessor](https://github.com/hww/lx-gpu-rtl)** — SystemVerilog RTL implementation of a programmable 2D graphics coprocessor, accompanied by a C++ functional emulator and microcode toolchain. `[RTL]` `[Verilator]` `[Emulation]`

- **[KIRA Neon](./kira_neon/)** — Compact, portable, low-cost synthesizer built on a custom hardware DSP platform. The proprietary sound synthesis firmware was developed by our team; my primary contribution to this project was hardware platform development. `[DSP]` `[SCH]` `[PCB]` `[SI/PI]` `[Firmware]`

- **[KIRA SHARC DSP Module](/projects/kira_dsp_module/index)** — Design of a standalone digital signal processing module, including hardware architecture and PCB development. The accompanying software was developed by other contributors; firmware development remained at an early stage. `[DSP]` `[SCH]` `[PCB]` `[SI/PI]` `[Firmware]`

- **[Aleste LX FPGA Computer](https://github.com/hww/aleste-lx-fpga-sv)** — FPGA implementation of a classic computer architecture using SystemVerilog, with Verilator-based simulation and verification. `[RTL]` `[Verilator]` `[Emulation]` `[Python]`

- **[Aleste XI FPGA Computer](/projects/xialeste)** — An attempt to create a highly accurate hardware implementation of the Aleste EX computer architecture. `[SCH]` `[PCB]` `[SI/PI]` 

- **[NuBridge — RISC-V NuBus Controller](https://github.com/hww/nubridge)** — Verilog implementation of a NuBus controller, including arbiter, master, and slave components, with SystemVerilog testbenches for RISC-V (PicoRV32) integration. `[RTL]` `[Verilator]`

- **[Rocket Amusements Cabinet I/O Board v2](/projects/ra_io_board2)** and **[v1](/projects/ra_io_board1)** — Development of arcade cabinet I/O hardware and peripheral interfaces. My contribution included hardware architecture, I/O design, technical specifications, and supporting code for firmware development. The main firmware implementation was carried out in collaboration with another developer. `[Firmware]` `[SCH]` `[PCB]`

- **[Nintendo GBA SP & DS Video Hardware](/projects/view_boy/)** — Custom hardware for capturing and outputting video from handheld game consoles. `[SCH]` `[PCB]` `[OPTICAL-DESIGN]`

---

## 2. Hardware & Firmware

Projects involving both electronic hardware and embedded firmware, where firmware development is a significant part of the engineering work.

- **[Digital Signal Processing & Real-Time Audio](/projects/remis/)** — DSP-based audio processing and real-time sound synthesis, combining hardware and embedded software. `[DSP]` `[SCH]` `[PCB]` `[RTL]` `[Firmware]` `[Software]`

---

## 3. End-to-End Hardware–Software Systems

Integrated projects covering multiple stages of product development: initial concept, system architecture, electronics, PCB design, embedded firmware, application software, and physical integration.

These projects demonstrate the ability to develop complete hardware-software systems rather than individual components in isolation.

- **[6DOF IMU Hand Tracking Gloves](/projects/ar_vr_gloves)** — End-to-end development of a wearable VR input device based on distributed 6DOF IMU sensors.

  My work covered the initial concept, system architecture, electronic schematics, PCB design, enclosure and mechanical integration, embedded firmware, sensor-fusion algorithms, and application software.

  Textile construction and pattern development were carried out with a specialist based on the mechanical and functional requirements of the device. `[Software]` `[Firmware]` `[SCH]` `[PCB]`

- **[Medical PPG Wristband Prototype](/projects/ppg/index)** — Development of a biometric sensing prototype combining electronic design, PCB layout, signal integrity considerations, embedded firmware, and supporting software for data acquisition and visualization. `[SCH]` `[PCB]` `[SI/PI]` `[Firmware]` `[Software]`

- **[Oculus Quest 2 Haptic Feedback Rifle](/projects/ar_vr_rifle)** — Development of the electrical system, custom I/O signaling, and mechanical interface for a haptic-feedback rifle peripheral used in commercial VR experiences. `[Software]` `[Firmware]` `[SCH]` `[PCB]`

- **[Nintendo 64 Hardware Debugger](/projects/n64_debugger)** — Hardware-assisted console debugging system, combining custom hardware interfaces with supporting software tools. `[SCH]` `[PCB]` `[RTL]` `[Software]`

---

## 4. Software, Emulation & Engineering Tools

Software projects, emulators, development tools, and low-level programming work related to hardware platforms, without requiring the development of a custom physical electronic device.

- **[Aleste 520 Computer Emulator](/projects/caprice32/)** — Software emulation of legacy computer hardware. `[Software]`

- **[Nintendo 64 Mini-Game](/projects/n64_game)** — Low-level software development for the Nintendo 64 platform. `[Software]`

- **[Aleste LX Platform & BIOS Development Kit](https://github.com/hww/aleste-xi-bios)** — Virtual 8-bit hardware platform, Z80 assembly BIOS, modular driver ABI, and GDB-enabled emulator. `[Emulation]` `[Software]`

- **[Buildex — Hardware Architecture Documentation Generator](https://github.com/hww/buildex)** — Python-based engineering tool for generating SVG architecture diagrams, register maps, and validated Markdown specifications from YAML descriptions. `[Software]` `[Python]`

- **[Magic Sound STM Player — Z80 Disassembly](https://github.com/hww/magic-sound-player)** — Reverse-engineered and annotated Z80 assembly source of a 1993 four-channel DMA audio tracker player for the Aleste 520EX. `[Software]` `[IDA-Pro]

- **[Crazy Claw Emojis Arcade Game](/projects/crazy_claw_emojis)** — Software development for a commercial arcade game. `[Software]` `[Unity]` `[Golang]`

- **[Crazy Claw Rings](/projects/crazy_claw_rings)** — Physics-driven arcade skill game developed for the amusement industry. `[Software]` `[Unity]` `[Golang]`

- **[Crazy Claw Junior](/projects/crazy_claw_jr)** and **[Crazy Claw Original](/projects/crazy_claw_original)** — Commercial arcade game software development. `[Software]` `[Unity]` `[Golang]`

---

## 5. Mechanical Design

- **[Pro Action Rocky Cheat Device](/projects/nintendo_cheat_device)** — Hardware interaction and low-level firmware development for a custom console peripheral. `[SolidWorks]` 

---

## Related Resources

- `[Articles & Technical Papers](/articles/)
- `[About Me](/about/)
- `[English CV](/resume.en)
- `[Резюме на русском](/resume.ru)
- `[GitHub repositories](https://github.com/hww)

[← Return to the main portfolio](/)
