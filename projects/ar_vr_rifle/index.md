# The haptic feedback rifle for Oculus Quest

My task was to design and prototype a plastic housing for the Oculus Quest controller, and this housing should contain a vibration motor for haptic feedback. There most complex part was to create the low cost, easy to build, and efficient vibration motor. After five different prototypes of the motor, was chose a MMA linear motor. All motors were simulated with the COMSOL, then their characteristics where estimated by nature test.

The images below displays two of the motors from all created prototypes.
<div>
<img src="/projects/ar_vr_rifle/images/motor_1.jpg" width="300" ><img src="/projects/ar_vr_rifle/images/vca_prototype.jpg" width="300" >
</div>

My task list for this project looks like:

- Designing and simulation the motor with COMSOL
- Designing the schematic and PCB with Siemens Xpedition
- Verifying the power integrity of PCB with Hyperlynx
- Designing the 3D model of the motor and the plastic housing with Siemens NX
- Printing a plastic housing
- Assembling
- Creating the firmware

Were madden 3 prototypes of the rifle. The photo of one of them you can see below.

<img src="/projects/ar_vr_rifle/images/rifle_03.jpg" width="500" >

The electonic circuit contains two DC/DC converters, the ESP32 microcontroller and L6207 motor driver. The PCB of the controller illustrated on the picture below.

<img src="/projects/ar_vr_rifle/images/vr_rifle_pcb.png" width="500" >

The firmware contains the UDP server and Lisp dialect interpretator. Each UDP request recognzed as string anf parsed by the LISP machine. To simplify integration to the Unity, there are LISP methods which interpolate the Unity.AnimationCurve for the single shoot or for the automatic shoots. 

*Technical characteristics*
|-----------------|------------|
| Тип вибромотора	| custom MMA |
| Максимальная частота выстрелов | 	20 Hz |
| Максимальная частота вибраций	| 50 Hz |
| Частота резонанса	| 27 Hz |
| Батарея	 | 12V 2Ah|
| Время работы от одной зарядки	| > 4h |
| Wi-Fi	| 2.4 GHz| 
| Масса	| 1.5 kg |
| Габариты	| 425x50x210 mm | 
| Интерфейс для диагностики и конфигурации	| Micro USB |
