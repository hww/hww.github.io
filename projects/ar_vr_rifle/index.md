# The haptic feedback rifle for Oculus Quest

*The brief product description*
The product is a prototype rifle with haptic feedback for Oculus Quest. The rifle has a quick-release left or right Oculus Quest controller mount. The controller is used to track the position and orientation of the rifle, and to control the game using the controls. The rifle housing contains a haptic feedback mechanism, a rapidly replaceable lithium battery, and a Wi-Fi enabled microcontroller. The microcontroller generates vibration style patterns based on commands over a wireless connection, using a protocol that can be easily integrated into any game.
 
*Technical characteristics*

|                 |            |
|-----------------|------------|
| Motor type    | custom MMA |
| Maximum shots per second |     20 Hz |
| Maximum vibration frequency  | 50 Hz |
| The 1st resonant frequency  | 27 Hz |
| Battery    | 12V 2Ah|
| Battery life time | > 4h |
| Wi-Fi    | 2.4 GHz|
| Mass   | 1.5 kg |
| Dimensions   | 425x50x210 mm |
| Diagnostic interface  | Micro USB |

*How i did it*
My task was to design and prototype a plastic housing and haptic feedback motor for the Oculus Quest controller. The most complex part was to create the low cost, easy to build, and efficient motor. After five different prototypes of the motor, the winner was an  MMA style linear motor.  All motors were simulated with the COMSOL, then their characteristics were verified by test.
The images below display two of the five prototype motors.
<div>
<img src="/projects/ar_vr_rifle/images/motor_1.jpg" width="300" ><img src="/projects/ar_vr_rifle/images/vca_prototype.jpg" width="300" >
</div>

My task list for this project included:

- Designing and simulating the motor with COMSOL
- Designing the schematic and PCB with Siemens Xpedition
- Verifying the power integrity of PCB with Hyperlynx
- Designing the 3D model of the motor and the plastic housing with Siemens NX
- Printing the plastic housing
- Assembling the prototypes
- Creating the firmware with C++ 17

Three different prototype  rifles  were made. One of them is shown in this photo:
<img src="/projects/ar_vr_rifle/images/rifle_03.jpg" width="500" >
The final prototype produces 5 N/A force and can produce an impact effect at up to 20 shots per second. The vibration effect is usable up to 50Hz. The rifle uses a 12V 2A/h quick-replace Metabo power tool battery.
The electronic circuit contains two DC/DC converters, an ESP32 microcontroller and an L6207 motor driver. The PCB of the controller is illustrated in the picture below.

<img src="/projects/ar_vr_rifle/images/vr_rifle_pcb.png" width="500" >

The firmware contains a UDP server and LISP based interpreter. Each UDP request is recognized as a string and parsed by the LISP machine. To simplify integration with Unity, there are LISP methods to interpolate Unity.AnimationCurve data for single shot and  automatic shot modes.

When the game runs and the Oculus controller sends a command to shoot, the game should send the command to the microcontroller in the rifle, and the microcontroller produces vibration or impact feedback.
Designing an electromagnetic system was a really tough  but  enjoyable task for me. 
Altogether, this was a very interesting project, and I am grateful to the company who requested this job task.

