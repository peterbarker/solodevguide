# Architectural Overview

Solo is a Linux system (iMX.6 running [Yocto Linux](advanced-linux.html)) connected to a Pixhawk autopilot.

The Pixhawk controls flight modes, stabilization, and recovery in the case of an RTL event (return-to-launch). Pixhawk communicates over the MAVLink telemetry protocol to both the on-board Linux computer and downstream devices like the Controller and mobile phone Solo apps.

The Linux system controls high-level operation of the copter: [smart shots](concept-smartshot.html), camera and gimbal control, mobile app communication, and accessory interaction are all implemented in this layer.

## Solo System Diagram

The diagram below is a simplified view of the main elements of the Solo architecture.

<aside class="note">This diagram is simplified in order to make it easier to understand the broad scope of the system, and to highlight the main integration points for "Made for Solo" accessories. Some components and systems that are relevant only to internal developers are not shown.</aside>

<img src="images/system-diagram.png" alt="Solo System Diagram" width="750" style="margin: 0 auto; display: block">

Data lines shown in solid blue with arrows showing the direction of data flow. Power lines are shown in red. Dashed-blue data lines to payload bays indicate data lines that are not available/supported for third-party use.

<!-- Diagram source is on Lucid here: https://www.lucidchart.com/documents/edit/61d4dfb6-701a-45f1-9525-a75a0d9fc8d5# -->


### Peripheral Mapping


#### Pixhawk2 connections
| PH2 Port | Connects to               | Role          | NuttX        | ArduCopter Parameter  |
|----------|---------------------------|---------------|--------------|-----------------------|
| USB      | Companion* (/dev/ttyACM0) | HAL/uartA     | /dev/ttyACM0 |                       |
| GPS      | Solo GPS                  | HAL/uartB     | /dev/ttyS3   | SERIAL3               |
| Telem1   | Companion (/dev/ttymxc1)  | HAL/uartC     | /dev/ttyS1   | SERIAL1               |
| Telem2   | Accessory Bay             | HAL/uartD     | /dev/ttyS2   | SERIAL2               |
| Serial4  | Gimbal                    | HAL/uartE     | /dev/ttyS6   | SERIAL4               |
| Serial5  | Accessory Bay             | nsh console   | /dev/ttyS5   |                       |
| SPKT/DSM | Companion (/dev/ttymxc2)  | DSM           |              |                       |
|          | PX4IO board               | PX4IO Console | /dev/ttyS0   |                       |
| CAN      | Accessory Bay             |               | ?            |                       |
| i2c      | Solo Compass              |               |              |                       |
|   i2c    | LED Controllersx4         |               |              |                       |
|   i2c    | Battery Controller Board  |               |              |                       |

#### Companion connections
| Device        | Connects to        | Role                   |
|---------------|--------------------|------------------------|
| /dev/ttymxc1  | PixHawk (Telem1)   | Telemetry              |
| /dev/ttymxc2  | PixHawk (DSM)      | DSM                    |
| /dev/ttymxc3  | ?                  | getty                  |
| USB Hub       | PixHawk OR Gimbal  |                        |
| USB Hub       | Accessory Bay      | Your Application here! |

<aside class="note">* A MUX is present which switches the Companion computer's USB connection between Pixhawk2 and the gimbal bay.  By default, this is connected to the gimbal bay.</aside>

## Solo hardware/software stack

* [3DR Poky](advanced-linux.html) (based on Yocto Project Reference Distro, Poky 1.5.1)
* Python 2.7.3
* ARM Cortex A9 ([i.MX6 Solo](http://www.freescale.com/products/arm-processors/i.mx-applications-processors-based-on-arm-cores/i.mx-6-processors/i.mx6qp/i.mx-6solo-processors-single-core-multimedia-3d-graphics-arm-cortex-a9-core:i.MX6S) by Freescale), 1Ghz, 1 CPU core with VPU and GPU
