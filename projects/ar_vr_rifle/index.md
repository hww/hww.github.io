# The haptic feedback rifle for Oculus Quest

My task was to design and prototype a plastic housing for the Oculus Quest controller, and this housing should contain a vibration motor for haptic feedback. There most complex part was to create the low cost, easy to build, and efficient vibration motor. After five different prototypes of the motor, was chose a MMA linear motor. All motors were simulated with the COMSOL, then their characteristics where estimated by nature test.

My task list for this project looks like:

- Designing and simulation the motor with COMSOL
- Designing the schematic and PCB with Siemens Xpedition
- Verifying the power integrity of PCB with Hyperlynx
- Designing the 3D model of the motor and the plastic housing with Siemens NX
- Printing a plastic housing
- Assembling
- Creating the firmware

Were madden 3 prototypes of the rifle. The photo of one of them you can see below.

<img src="/projects/ar_vr_rifle/images/rifle_03.jpg" width="500" c

The electonic circuit contains two DC/DC converters, the ESP32 microcontroller and L6207 motor driver. The PCB of the controller illustrated on the picture below.

<img src="/projects/ar_vr_rifle/images/vr_rifle_pcb.png" width="500" >

<img src="/projects/ar_vr_rifle/images/pcb_block_diagramm.png" width="300">

<img src="/projects/ar_vr_rifle/images/vca_prototype.jpg" width="300" >



