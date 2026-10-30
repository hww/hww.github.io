# KIRA SHARC — Digital Signal Processor Module

*An embedded audio computing module based on SHARC processors*

## Purpose and Form Factor

KIRA DSP is a compact embedded digital signal processing module designed for integration into modern digital and digital-analog equipment. Its primary applications are in professional audio equipment and hardware sound synthesizers.

The module features an ultra-compact DDR2 SODIMM form factor. This minimizes the footprint of the host device's motherboard and allows for quick replacement or upgrades of the computing module.

![Aleste 520EX](/projects/kira_dsp_module/pcb.jpg)

## Processing Core and Memory

- **Processor:** High-performance ADSP-21584 dedicated audio DSP from Analog Devices.
- **RAM:** 2 Gbit of DDR3 memory with a 16-bit data bus. The memory bus operates at 400 million transfers per second.
- **Non-volatile memory:** High-speed Quad SPI Flash memory for storing firmware and user-defined algorithms.

## Synchronization and Clocking

To achieve minimal jitter and the highest possible audio processing quality, the board features two high-quality, highly stable crystal oscillators:

- **49.152 MHz** — A dedicated audio clock frequency, aligned with the 48 kHz sampling-rate family.
- **24.000 MHz** — For clocking system peripherals.

The processor's internal operating frequency can be flexibly configured through software.

## Peripherals and Interfaces

All key signals are routed to the module's edge connector, providing developers with access to the following interfaces:

- **Audio interfaces:** All signals of the dedicated DAI (Digital Audio Interface) bus.
- **Communication interfaces:** USB, SPI, and four two-wire interfaces.
- **General-purpose I/O:** 16-bit GPIO port.
- **Debugging:** Dedicated JTAG interface for programming and debugging.

## Power Supply and Status Indicators

- **Power supply:** The module operates from a single external 5 V power source.
- **Voltage regulation:** High-efficiency switching buck regulators are integrated directly onto the board. They provide independent power rails for the processor core, peripherals, and memory.
- **Monitoring and control:** The board features hardware jumpers for selecting the boot mode, along with LED status indicators for:
  - Power supply status
  - Normal operation
  - Failure indication (FAIL)

## PCB Stackup and Signal Integrity

The six-layer PCB stackup proved to be close to its practical limits. The board design provides only two pairs of closely spaced layers, significantly constraining the available routing options and limiting flexibility in impedance control and return-path management. Furthermore, the top layer has limited routing capacity due to the dense placement of the BGA package and other SMD components, leaving little room for critical signal traces.

Signal integrity and power integrity analyses were performed using Siemens HyperLynx to verify the design and assess its electrical performance.