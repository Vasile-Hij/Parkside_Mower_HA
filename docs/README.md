# Parkside_Mower_HA
Connect a non-smart robot lawn mower to Home Assistant, using ESPHome.

Running on a **Parkside PAMR 500 A1**. Based on [Gabriel-LG/HA_Mower](https://github.com/Gabriel-LG/HA_Mower), extended with battery protection, night quiet mode, theft detection, 2 zones and HA automations.\
![image](user_interface.png)

## Files
| File | What it is |
|---|---|
| `Robot_mower.yaml` | ESPHome, ESP8266 (Wemos D1 mini) inside the mower |
| `Mower_zone.yaml` | ESPHome, zone relay (ESP-01 + relay + BME280) at the dock |
| `home_assistant/automations.yaml` | HA automations |
| `home_assistant/helpers.yaml` | HA helpers (timer, input_text, `Mower state` sensor) |

Secrets go in ESPHome `secrets.yaml`. Change the fallback hotspot passwords (`xyz`).

## Concept
Mower setup: **1 zone, schedule 00:00 - 23:59 every day, stop on rain and resume.**

The ESP fakes the rain sensor: `Mow` OFF = rain (stays docked), `Mow` ON = dry (mows).\
HA decides **when** (`Mow` + `Mowing time`) and **where** (`Mower zone`). The mowing timer runs on the ESP, so the mower returns even without WiFi or HA.

## 2 zones
![image](two_zones.svg)

A relay switches the dock's wire output between two loops: **Zone 1** = relay ON (NO), **Zone 2** = relay OFF (NC, default). Change zone only while docked.

## Mower ESP features
- Status: `Standby`, `Preparing`, `Mowing`, `Returning`, `Fault` (not mowing, moving or charging), `Unknown`
- Charging detected from battery voltage
- Low battery (18.25 V): stops and blocks mowing. Critical (17.0 V): deep sleep
- Quiet mode: WiFi off while docked, on for minutes :00-:03 of every hour
- Theft alarm while docked (shake or tilt)
- Start while offline: HA sets `input_boolean.mower_start_pending`, picked up at next check-in

The mower charges only after a run; docked, the ESP drains it about 0.7 V/day.

## Home Assistant
- **Mower - simple logic**: Mon / Wed / Fri 11:11 and 15:11, starts mowing
- **Mower - start / cancel countdown**: dashboard timer
- **Mower - last check**: keeps last values while the mower is offline
- `sensor.mower_state`: `Working`, `Sleeping - charging OK`, `Idle - charging OK`, `Not charging`, `Warning` (offline 2 h), `Issue` (offline 10 h)

## Building the hardware
ESP8266 board, accelerometer board, DC/DC converter and 3 resistors on a piece of perfboard.\
![image](assembled.jpg)\
![image](circuit.png)

Mounted on the battery pack and soldered to the battery poles (inside the battery housing). In the robot, one wire is spliced into a rain sensor wire.\
![image](installed.jpg)

Upload `Robot_mower.yaml` to the mower ESP and `Mower_zone.yaml` to the zone relay. Make sure there is WiFi coverage on the lawn, or at least at the dock. Other mowers may need different battery values in `substitutions`.

## Credits
PAMR 500 A1 version, 2 zones and Home Assistant automations: [Vasile Hij](https://github.com/Vasile-Hij).\
Original project and hardware design: [Gabriel-LG/HA_Mower](https://github.com/Gabriel-LG/HA_Mower).
