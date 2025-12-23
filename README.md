# RoadMarkerBot

An open-source road marking bot with

- skid steering
- differential GPS (Base + Rover)
- Ardupilot
- multiple colors
- dual use as a survey tool.

## Architecture

### System Architecture

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

### Electrical Design

#### Rover

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

#### RTK Base Station

```mermaid
graph TD
    %% Power Source
    PB(USB Power Bank) -- "5V via USB" --> BASE_GPS(RTK GPS Base Module)

    %% Internal Connection
    BASE_GPS -- "UART (RTCM3x)" --> RADIO(Radio Module<br>on Base)

    %% Wireless Output
    RADIO -. "868 MHz RTCM Corrections" .-> ROVER(Radio Module<br>on Rover)
```

#### Survey Stick

```mermaid
graph TD
    
    %% Communication & Interface
    subgraph Handheld_Unit [Survey Tool]
        ESP
        BTN(Push Button)
        GPS_ROV(GPS from Rover)
        RADIO(Radio Module<br>from Rover)
    end
    ESP(ESP8266)

    %% Data Connections
    RADIO -- "UART (RTCM3x)" --> GPS_ROV
    GPS_ROV -- "UART (NMEA)" --> ESP
    BTN -- "Trigger" --> ESP

    %% External Links
    BASE(RTK Base Station) -. "868 MHz RCTM Corrections" .-> RADIO
    ESP -- "UART (NMEA) via USB" --> SERVER(Mobile Device)
```

### Mechanical Design

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

Detailed images of the rover can be found [HERE](./rover/gallery.md).

Detailed images of the base station can be found [HERE](./base/gallery.md).

Detailed images of the survey tool can be found [HERE](./survey_tool/gallery.md).

#### Config Files

A detailed list of the config files needed for the rover can be found [HERE](./rover/config.md).

A detailed list of the config files needed for the base station can be found [HERE](./rover/config.md).

A detailed list of the config files needed for the survey tool can be found [HERE](./rover/config.md).

A detailed list of config files and parts for the Ground Control Station can be found [HERE](./gcs/readme.md)
