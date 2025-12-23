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
    subgraph PIXHAWK [Pixhawk Flight Controller]
        CPU(Processor / ArduPilot / PX4)
        IMU_INT(Internal Gyro / IMU)
    end

    %% Sensors & Inputs
    IMU_EXT(External Gyro / IMU) -- "I2C Bus" --> PIXHAWK
    GPS_ROVER(RTK GPS Rover) -- "UART" --> PIXHAWK
    RC_REC(RC Receiver) -- "UART" --> PIXHAWK
    TELE(Telemetry Module) -- "UART (MAVLink)" --> PIXHAWK
    
    %% Power Monitoring (New)
    PWR_SENS(Power Sensor) -- "Analog (V/I Data)" --> PIXHAWK

    %% Wireless Links
    GCS(Ground Control Station) -. "868 MHz Radio" .-> TELE
    Remote(Remote Control) -. "RC Link" .-> RC_REC
    
    subgraph RTK_Base_System [External Reference]
        GPS_BASE(RTK GPS Base Station)
    end
    GPS_BASE -. "868 MHz RTCM Corrections" .-> GPS_ROVER

    %% Actuators (Outputs)
    PIXHAWK -- "PWM" --> ESC(ESC Motor Controller)
    ESC --> MOT(2x Drive Motors)
    
    PIXHAWK -- "PWM" --> SER(4x Servos)

    %% Styling
    classDef hardware fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef external fill:#fff9c4,stroke:#fbc02d;
    classDef rtk fill:#e8f5e9,stroke:#2e7d32;
    class PIXHAWK hardware;
    class IMU_EXT,GPS_ROVER,RC_REC,TELE,PWR_SENS external;
    class GPS_BASE rtk;
```

### Electrical Design

#### Rover

```mermaid
graph TD
    %% Power Source and Monitoring
    BAT(4S LiPo Battery<br/>14.8V - 16.8V) --> SENS(Power Sensor / <br/>Current & Voltage)
    
    %% Monitoring Link
    SENS -- "Analog Data (V/I)" --> PH(Pixhawk Flight Controller)

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

<details>
<summary>ROVER: Images</summary>
![frame_iso](rover/bare_frame_iso.png)
![frame_top](rover/bare_frame_top.png)
![wheel_assembly](rover/wheel_assembly.png)
![third_wheel_top](rover/third_wheel_top.png)
![third_wheel_bottom](rover/third_wheel_bottom.png)
![sprayer_assembly](rover/sprayer_assembly.png)
![sprayer_assembly_bottom](rover/sprayer_assembly_bottom.png)
![sprayer_assembly_bottom_nocover](rover/sprayer_assembly_bottom_nocover.png)
![mounts_for_electronics](rover/mounts_for_electronics.png)
![full_model](rover/full_model.png)
</details>

<details>
<summary>BASE: Images</summary>
ROVER: Pixhawk Config
</details>

<details>
<summary>SURVEY TOOL: Images</summary>
ROVER: Pixhawk Config
</details>

#### Config Files

<details>
<summary>ROVER: Pixhawk Config</summary>
ROVER: Pixhawk Config
</details>

<details>
<summary>ROVER: uBlox F9P Config</summary>
ROVER: uBlox F9P Config
</details>

<details>
<summary>ROVER: ESC Config</summary>
ROVER: ESC Config
</details>

<details>
<summary>BASE: uBlox F9P Config</summary>
BASE: uBlox F9P Config
</details>

<details>
<summary>SURVEY TOOL: ESP Config</summary>
SURVEY TOOL: ESP Config
</details>
