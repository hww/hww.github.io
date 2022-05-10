# Hand tracking gloves
*A brief product description*
The project was to create  and prototype hand tracking gloves for VR. High tracking accuracy and interference protection allow the gloves to be used for  industrial environments and processes. The software integrates easily into modern gaming platforms. The use of a quick-release battery, wireless operation, and adjustability for a quick custom fit  provide convenience, reliability, accurate use, comfort, and the most vivid user experience. Battery and Wi-Fi signal status are indicated by both LED and tactile alarms. The glove line consists of two models: Elite and Professional. The professional version is designed for industrial use with a larger battery providing more run time and more processor performance.

<div>
<img src="/projects/ar_vr_gloves/images/proto_3_1.jpg" width="300" >
<img src="/projects/ar_vr_gloves/images/proto_3_3.jpg" width="300" >
</div>


*The specifications*

|                                              | Elite     | Professional |
|----------------------------------------------|-----------|--------------|
| Calibrating time                         | 4-5 m     | 4-5 m        |
| Sensor reading frequency              | up to 400 Hz    | up to 800 Hz       |
| Host computer update rate by Wi-Fi       | 30 fps    | 60 fps       |
| Host computer update rate by wire        | 60 fps    | 120 fps      |
| Battery capacity                         | 500 mA/h  | 800 mA/h     |
| Battery life time                        | 5 h       | 8 h          |
| Battery life time in idle           | 48 h      | 48 h         |
| Battery charge time                      | 40 m      | 1 h          |

*How I did it*

The task was to design and prototype hand tracking gloves. The product should be low cost, easy to produce, with precise, high-speed tracking. The gloves should be comfortable to wear and easy to put on and remove.
I built four prototypes with different types of IMU sensors and with different numbers of sensors in different positions on the hand.

All prototypes shared the same control circuit, which includes:
- Power management and battery charger
- ESP32 microcontroller
- USB <-> Serial interface

<div>
<img src="/projects/ar_vr_gloves/images/pcb.jpg" width="300" >
</div>

My task list for this project included:

- R&D Research all existing products and choose best possible design
- Design the schematic and PCB for microcontroller and sensors with Siemens Xpedition
- Design the plastic housing with Siemens NX
- Create the firmware with C++
- Create the host software and Unity 3D integration

The firmware contains a UDP server, and various filters: Kalman, Mahony, Madgwick.
The main calibrating algorithms are implemented on the host side. The table below contains
the sensors used in the prototypes. All of them have approximately the same result, and there are no winners.

*The sensors used in prototypes*

| Manufacturer  | Part ID   |
|---------------|-----------|
| BOSH          | BMX055    |
| ST            | LSM9DS1   |
| TDK           | ICM-20948 |
| BOSH          | BMX160    |

The prototypes used the same battery 800 mA/h, but the testing showed it was too large. There is an opportunity to create a reduced version of the product with a smaller battery. 
This  was a very interesting project, and I am grateful to the company who requested this work.
