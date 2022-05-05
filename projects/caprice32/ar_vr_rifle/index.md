
# The rifle with haptic feedback for Oculus Quest

The plastic housing for Oculus Quest II controller, contains a microcontroller board and custom MMA linear motor. The Metabo's 12V 2A/h battery used as power source.  The power circuit includes two DC/DC converters, one for the MCU and second for a motor driver L6207. 

After boot, the microcontroller create an access point or station connection. Then it starts UDB server, and parse all request with one of LISP dialect interpretator. There valious functions to conttroll the motor for procdcing diffferent style of the haptic feedback. Those functions could use the Unity's AnimationCurrve to change the motor's current.

The custom MMA motor is most complecx part of this project. There was requirement to make low cost and weight, 38mm diameter, and about 5N/A efficiency motor. The modeling was done with the COMSOL. I have to create 5 different motors for select the best wich was used for final design.

Schematic circuit had 3 different prototype, and last one with ESP32 and L6207 was choosed as most simple and flexible. The schematic circuit and PCB layout created with Siemens Xpedition. The PI modeling and verification with MentorGraphics Hyperlinx. 

The plastic housing designed with Siemens NX and printed with PETG.



