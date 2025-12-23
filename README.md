# RoadMarkerBot
An open-source road marking bot with 
- skid steering
- differential GPS (Base + Rover)
- Ardupilot
- multiple colors.

## Architecture

### System Architecture

```mermaid
graph TD
    %% Central Unit
    subgraph PIXHAWK [Pixhawk Flight Controller]
        CPU[Processor / ArduPilot / PX4]
        IMU_INT[Internal Gyro / IMU]
    end

    %% I2C Bus
    IMU_EXT[External Gyro / IMU] -- "I2C Bus" --> PIXHAWK

    %% UART Interfaces
    GPS_ROVER[RTK GPS Rover] -- "UART" --> PIXHAWK
    RC_REC[RC Receiver] -- "UART" --> PIXHAWK
    TELE[Telemetry Module] -- "UART (MAVLink)" --> PIXHAWK

    %% Wireless Links
    GCS[Ground Control Station] -. "868 MHz Radio" .-> TELE
    Remote[Remote Control] -. "RC Link" .-> RC_REC
    
    %% RTK Specific Link
    subgraph RTK_Base_System [External Reference]
        GPS_BASE[RTK GPS Base Station]
    end
    GPS_BASE -. "868 MHz RTCM Corrections" .-> GPS_ROVER

    %% Actuators (Outputs)
    PIXHAWK -- "PWM" --> ESC[ESC Motor Controller]
    ESC --> MOT[2x Drive Motors]
    
    PIXHAWK -- "PWM" --> SER[4x Servos]

    %% Styling
    classDef hardware fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef external fill:#fff9c4,stroke:#fbc02d;
    classDef rtk fill:#e8f5e9,stroke:#2e7d32;
    class PIXHAWK hardware;
    class IMU_EXT,GPS_ROVER,RC_REC,TELE external;
    class GPS_BASE rtk;
```

### Electrical Design

### Mechanical Design

### Resources

#### BOM

| Rover (Mechanics) | Rover (Electrics) | Misc |
| ----------------- | ----------------- | -----|
| | | |
| | | |
| | | |
| | | |
| | | |
| | | |
| | | |

#### Config Files
