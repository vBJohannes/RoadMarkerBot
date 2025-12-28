# GPS Base Station

## Architecture

```mermaid
graph TD
    %% Power Source
    PB(USB Power Bank) -- "5V via USB" --> BASE_GPS(RTK GPS Base Module)

    %% Internal Connection
    BASE_GPS -- "UART (RTCM3x)" --> RADIO(Radio Module<br>on Base)

    %% Wireless Output
    RADIO -. "868 MHz RTCM Corrections" .-> ROVER(Radio Module<br>on Rover)
```

## Design

The GPS base station is a rather simple setup and was put together from remaining parts and simple 3D printed parts in order to make a solid yet reliable device.
It is about 1000mm in height (to get the antenna off the ground without being too susceptable to wind) and the metal shield at the top has an excessive 300mm diameter (to reduce reflections as much as possible).
It is powered by a regular powerbank and will run for quite some time on it as is only draws about XXXmA of current when sending correction data.
The antenna is put to the corner of the upper shield to be also elevated off the ground.

![gps_base1](./img/gps_base1.png)
![gps_base2](./img/gps_base2.jpg)
![gps_base3](./img/gps_base3.jpg)
![gps_base_detail1](./img/gps_base_detail1.jpg)
![gps_base_detail2](./img/gps_base_detail2.jpg)

# Usage

- The antenna is placed on a dedicated spot and the ground is marked so the antenna can be placed there repeatedly in the exact same configuration.
- Ideally the spot is surveyed in by using the antenna, a laptop, some free Ntrip data service like [RTK2Go](http://rtk2go.com), and the rover GPS config (see [Rover GPS Config](ROVER_LINKS)) to figure out the exact position the GPS is used in.
This can then be entered in the entered in the config of the GPS to run it in fixed mode.
- Both communication modules need to use the same config to communicate properly.

# Links

[GPS Base uBlox Config](./config/uBlox_F9P_Base_config.txt)<br>
[XBee RTCM Config](./config/GPS_RTCM_XBeeSX686_fwA00A.xpro)
