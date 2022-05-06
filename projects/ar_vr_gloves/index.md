# The hands tracking gloves

My task was to design and prototype a hands tracking gloves. The product should be low cost easy to produce and have a high precise of tracking.
I build four prototypes with differect type of IMU sensors and different quantity and positions of them on tha hand. 

All prototypes shared same electronics circuit, wich includes:

- Power management and battery charger
- ESP32 microcontroller
- USB <-> Serial interface

My job task list contains:

- R&D Research all existing products and choose best possible design
- Desing the schematic and PCB for microcontroller and sensors with Siemens Xpedition
- Design the plastic housing with Siemens NX
- Create the firmware with C++
- Create the host software and Unity 3D integration

The firmare contains UDP server, and various of filters: Kalman, Mahony, Madgwick. 
The main calibrating algorythms implemented on the host side. The table below contains
the sensors used in the prototypes. All of them have aproximately same result, and there are no any winner.

*The sensors used in prototypes*

| Manufacturer  | Part ID   |
|---------------|-----------|
| BOSH          | BMX055    |
| ST            | LSM9DS1   |
| TDK           | ICM-20948 |
| BOSH          | BMX160    |

The prototypes where used same battery 800 mA/h, but the testing showed
it is too large capacity. And there is possibility ti create a bit 
dowgraded version of product. The final specification for the prototypes
contains the data for dowgraded version (Elite), and real full
featured version (Professional).

*The specifications*

|                                              | Elite     | Professional |
|----------------------------------------------+-----------+--------------|
| The calibrating time                         | 4-5 m     | 4-5 m        |
| The sensor's reading frequencly              | up to 400 Hz    | up to 800 Hz       |
| The host computer update rate by Wi-Fi       | 30 fps    | 60 fps       |
| The host computer update rate by wire        | 60 fps    | 120 fps      |
| The battery capacity                         | 500 mA/h  | 800 mA/h     |
| The battery life time                        | 5 h       | 8 h          |
| The battery life time in the iddle           | 48 h      | 48 h         |
| The battery charge time                      | 40 m      | 1 h          |


That was very interesting project, and I grateful to the company who requuest this job task.
