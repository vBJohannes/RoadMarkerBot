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
    subgraph PIXHAWK [Pixhawk 2.4.8<br>Flight Controller]
        CPU(Processor / ArduPilot / PX4)
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
    GCS(Ground Control Station<br>Ardupilot Mission Planner) -. "868 MHz Radio" .-> TELE
    Remote(Remote Control<br>with EdgeTX) -. "ELRS Link" .-> RC_REC

    %% GPS RTK
    GPS_BASE(RTK GPS Base Station) -. "868 MHz RTCM Corrections" .-> GPS_ROVER

    %% Actuators (Outputs)
    PIXHAWK -- "PWM" --> ESC(ESC Motor Controller<br>AM32)
    ESC --> MOT(2x Drive Motors)
    
    PIXHAWK -- "PWM" --> SER(4x Servos<br>for Spray Cans)

    %% Styling
    classDef system fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef software fill:#e8f5e9,stroke:#2e7d32;
    classDef external fill:#fff9c4,stroke:#fbc02d;
    class PIXHAWK,GPS_BASE system;
    class GCS software;
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

    %% Styling
    style BAT fill:#f96,stroke:#333
    style SENS fill:#ffeb3b,stroke:#fbc02d,stroke-width:2px
    style PH fill:#cfd8dc,stroke:#455a64
    style REG1 fill:#e1f5fe,stroke:#01579b
    style REG2 fill:#e1f5fe,stroke:#01579b
```

#### RTK Base Station

```mermaid
graph TD
    %% Power Source
    PB(USB Power Bank) -- "5V via USB" --> BASE_GPS(RTK GPS Base Module)

    %% Data & Communication
    subgraph Base_Unit [Base Station Core]
        BASE_GPS
        RADIO(868 MHz Radio Module)
    end

    %% Internal Connection
    BASE_GPS -- "UART / Data" --> RADIO

    %% Wireless Output
    RADIO -. "868 MHz RTCM Corrections" .-> ROVER(Rover / Robot)

    %% Styling
    style PB fill:#f96,stroke:#333,stroke-width:2px
    style BASE_GPS fill:#e1f5fe,stroke:#01579b
    style RADIO fill:#fff9c4,stroke:#fbc02d
    style ROVER fill:#cfd8dc,stroke:#455a64,stroke-dasharray: 5 5
```

#### Survey Stick

```mermaid
graph TD
    %% Power Source
    PB(USB Power Bank) -- "5V via USB" --> ESP(ESP32 / ESP8266)
    
    %% Communication & Interface
    subgraph Handheld_Unit [Survey Tool Core]
        ESP
        BTN(Push Button / Trigger)
        GPS_ROV
        RADIO(868 MHz Radio Module)
    end

    %% Data Connections
    RADIO -- "RTCM Corrections" --> GPS_ROV
    GPS_ROV -- "UART (NMEA), 5V" --> ESP
    BTN -- "GPIO / Digital Input" --> ESP

    %% External Links
    BASE(RTK Base Station) -. "868 MHz Link" .-> RADIO
    ESP -. "WiFi (GPS Data Transfer)" .-> SERVER(Smartphone / Laptop / Server)

    %% Styling
    style PB fill:#f96,stroke:#333
    style ESP fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    style BTN fill:#ffc107,stroke:#333
    style GPS_ROV fill:#fff9c4,stroke:#fbc02d
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
