# Rover

## Logical Architecture

```mermaid
graph TD
    %% Central Unit
    subgraph PIXHAWK [Pixhawk 2.4.8]
        AP(ArduPilot)
        CPU(Processor)
        IMU_INT(Internal Gyro / IMU)
    end

    %% Sensors & Inputs
    IMU_EXT(External Gyro) -- "I2C Bus" --> PIXHAWK
    GPS_ROVER(RTK GPS Rover) -- "UART" --> PIXHAWK
    RC_REC(RC Receiver) -- "UART" --> PIXHAWK
    TELE(Telemetry Module) -- "UART (MAVLink)" --> PIXHAWK
    
    %% Power Monitoring (New)
    PWR_SENS(Power Sensor) -- "Analog (U/I Data)" --> PIXHAWK

    %% Wireless Links
    GCS(Ground Control Station<br>Ardupilot Mission Planner) <-. "868 MHz Radio" .-> TELE
    Remote(Remote Control<br>with EdgeTX) <-. "ELRS Link" .-> RC_REC

    %% GPS RTK
    GPS_BASE(RTK GPS Base Station) -. "868 MHz RTCM Corrections" .-> GPS_ROVER

    %% Actuators (Outputs)
    PIXHAWK -- "PWM" --> ESC(ESC Motor Controller<br>AM32)
    ESC --> MOT(2x BLDC Drive Motors)
    
    PIXHAWK -- "PWM" --> SER(4x Servos<br>for Spray Cans)
```

## Electrical Archtecture

```mermaid
graph TD
    %% Power Source and Monitoring
    BAT(4-6S LiPo Battery) --> SENS(Power Sensor / <br/>Current & Voltage)
    
    %% Monitoring Link
    SENS -- "Analog Data (U/I)" --> PH(Pixhawk Flight Controller)

    %% Direct Power Distribution
    SENS --> ESC(Dual ESC<br/>Motor Controller)
    SENS --> REG1(5V Voltage Regulator<br/>Logic Supply)
    SENS --> REG2(5V Voltage Regulator<br/>Servo Supply)

    %% Drive System
    ESC --> M1(Drive Motor Left)
    ESC --> M2(Drive Motor Right)

    %% Logic Supply Path
    REG1 --> PH
    PH --> GPS(RTK GPS Rover)
    PH --> TELE(Telemetry Module)
    PH --> RC(RC Receiver)
    PH --> GYRO(External Gyro / IMU)

    %% Servo Supply Path
    REG2 --> SERVO_BUS(Servo Rail)
    SERVO_BUS --> S_ALL(4x Servos)

    %% Control Signals (Simplified)
    PH -. "PWM" .-> ESC
    PH -. "PWM" .-> S_ALL
```

## Design

The robot was designed as a skid steer vehicle with a trailing third wheel.
An extensible design was chosen to allow quick modifications.
Overall vehicle dimensions: width 600 mm, length 500 mm, height 400 mm

Built with:

- 20×20 mm aluminium extrusions to mount various components
- high gear reduction on the motors for precise maneuvers
- simple power supply using 18 V batteries compatible with Makita or Bosch
- readily available hobby-model parts
- autonomous control via ArduPilot software
- four spray cans can be mounted and all spray to the same point

### Frame

![frame_iso](./img/bare_frame_iso.png)
![frame_top](./img/bare_frame_top.png)

### Wheel Assembly

For this project powerful and low-KV motors were used.
The purpose was to make small corrections with the high torque and low rpm.
In addition a reduction of about 45:1 was achieved by a gearbox (19:1) and pulleys (2.35:1).
This specific BLDC motor is a *Robbe 5052 410K/V* rated for 50A at 6S.
Due to the reduction the rover reaches speeds of about 5km/h.
The wheel itself is a solid rubber 190mm replacement wheel for e-scooters and was adapted to carry a pully.
All the parts can be found in [MISSING](./motor_wheel_parts.zip) archive.

![wheel_assembly1](./img/wheel_assembly1.png)
![wheel_assembly2](./img/wheel_assembly2.jpg)
![wheel_assembly3](.img/wheel_assembly3.png)
![third_wheel_top](./img/third_wheel_top.png)
![third_wheel_bottom](./img/third_wheel_bottom.png)

### GPS

On the rover and the base station a board with *uBlox F9P* module is used.
The particular part is an *ArduSimple simpleRTK2B* with *Radio Module Long Range (LR)* and *ANN-MB-00* antenna.
The GPS module is mounted below the connection PCB.
![gps_mount1](./img/gps_mount1.png))
![gps_mount2](./img/gps_mount2.jpg))
The step files for the base can be found [HERE](gps_mount_base.step) and the one for the retainer can be found [HERE](gps_mount_retainer.step).
The config file for the GPS can be found [HERE](MISSING) and can be applied using the [u-Center](LINK-MISSING).
The config file for the radio module can be found [HERE](GPS_RTCM_XBeeSX686_fwA00A.xpro) and should also be used on the [Base Station](base_station_gps).
The antenna is placed on a 200x200x1mm sheet of aluminium to filter out ground reflections.
![gps_antenna_rover1](./img/gps_antenna_rover1.jpg)
![gps_antenna_rover2](./img/gps_antenna_rover2.jpg)

### Spray Can Pods

The spray can pod is designed in a way, that all four cans mark the same spot right in the center of the (imaginary) front axle.
The pod is printed in one piece and is intended to make the frame more rigid.
The cans can be attached and detached after loosening the thumb screw.
This pod is designed around a chalk marking spray from Technima.
On the under side the trigger of the cans are activated by servos, which are in turn covered to prevent dirt and debris from damaging the rover while in operation.

![spray_can](./img/spray_can.jpg)
![pod1](./img/pod1.png)
![pod2](./img/pod2.png)
![pod3](./img/pod3.jpg)

### Power Supply

For this project the focus was availibility of power sources, so an adapter to run off batteries compatible with Bosch's 18V system.
Initially two batteries were planned, but due to very small power draw, the rover runs for several kilometers on a single battery.
Due to reverse current protection a battery can be added during operation and the other one can be pull from the rover without interrupting operation.
![battery_port1](./img/battery_port1.jpg)
![battery_port2](./img/battery_port2.jpg)

### Flight Controller

As flight controller a *Pixhawk 2.4.8* was used due to high availability at reasonable prices.
This comes at the cost of limited computational power when handling waypoints.
The flight controller has enough IO ports and can run ArduPilot, which was used for the project.
The config file for the flight controller can be found [HERE](pixhawk_config.txt).
In order to fix the flight controller to the frame two side supports are used.
The step file can be found [HERE](./pixhawk_mount.step).
It has a cutout for the USB port and as it is symmetric, it is needed twice.
![pixhawk_mount1](./img/pixhawk_mount1.png)
![pixhawk_mount2](./img/pixhawk_mount2.jpg)

### ESC

For this project a BLHELI32 compatible ESC was used due to its configurability.
The exact part was a *T-Motor P60A v2*, which proides 60A from 3-6S LiPo batteries.
To run the motors, this config was used on output M3 and M4.
For configuration a *Sequre ESC-Link* and the *BlHeli32 Suite* was used.
The ESC itself is mounted to the frame and also provides connectors for the motor wiring.
Both step files can be found [HERE](./esc_mount.step) and [HERE](./banana_plug_mount.step).
![esc_config](./img/BLHeli32_ESC_config.png)
![esc_mount1](./img/esc_mount1.png)
![esc_mount2](./img/esc_mount2.jpg)
![banana_mount1](./banana_plug_mount1.png)
![banana_mount2](./banana_plug_mount2.jpg)

<a id="connection_pcb"></a>

### Connection PCB

This board is intended to have a common connector for all sensors and actors connected to the Pixhawk.
This specifically applies to GPS, Servos, ELRS receiver, ESC, and the telemetry radio module.
It is also intended to be used to convert voltage from battery to 5V and 3.3V.
The KiCad file can be found [HERE](dotty2_adapter_board.zip) and the STEP file to hold down the radio module can be found [HERE](dotty2_adpater_board_radio_module_retainer.step).
![dotty2_adapter_pcb1](./img/adapter_pcb1.png)
![dotty2_adapter_pcb2](./img/adapter_pcb2.jpg)
![dotty2_adapter_board1](./img/adapter_board1.png))
![dotty2_adapter_board2](./img/adapter_board2.jpg))