**Crossing Gate ESP32 #1**

Obstacle Avoidance IR Sensor Implementation

Development Session Report

Cybersecurity of Digitized Railroad Track Control Simulation

LEGO-Based Model Railway with Raspberry Pi 4B & OpenPLC

Internship Project

**May 2026**

# 1. Session Overview

This report documents the complete development session for ESP32 #1 — the crossing gate controller using obstacle avoidance IR sensors. The session covered hardware wiring, firmware development and iteration, OpenPLC slave device configuration, and ladder diagram integration. The device controls a servo-operated crossing gate and RGB LED warning light, responding autonomously to train detection while being monitored and enabled/disabled by OpenPLC.

# 2. Hardware

## 2.1 ESP32 Board

This ESP32 uses a standard development board (HW-777) with header pins instead of screw terminals. The board has three GND pins, a 3.3V pin, and a VCC (5V) pin, allowing multiple components to connect to separate power and ground pins simultaneously.

## 2.2 Pin Assignment

| Component | Pin | Voltage | Notes |
| --- | --- | --- | --- |
| IR Sensor 1 OUT | GPIO 34 | 3.3V signal | Train approaching |
| IR Sensor 2 OUT | GPIO 35 | 3.3V signal | Train cleared |
| IR Sensors VCC | 3V3 | 3.3V | Both sensors share |
| IR Sensors GND | GND | 0V | Common ground |
| RGB LED Red pin | GPIO 26 | 3.3V signal | Only red pin used |
| RGB LED V pin | 3V3 | 3.3V | Power for LED |
| Servo signal | GPIO 25 | 3.3V signal | PWM control |
| Servo VCC | VCC | 5V | Needs 5V for torque |
| Servo GND | GND | 0V | Common ground |

## 2.3 RGB LED Module

The Keyestudio RGB LED module (KS0181) has four pins: G (green), R (red), B (blue), and V (VCC). Since only red blinking is needed for the crossing gate warning, only the R and V pins are connected. The G and B pins are left unconnected.

## 2.4 IR Sensor Logic

The obstacle avoidance IR sensor outputs LOW when it detects an object and HIGH when the path is clear. This is the opposite of some other sensor types. The firmware uses this logic:

bool `trainApproaching` = (`digitalRead``(``IR_SENSOR_1) == LOW``);  /``/ LOW = detected`

bool `trainCleared`     = (`digitalRead``(``IR_SENSOR_2) == LOW``);  /``/ LOW = detected`

# 3. Firmware Development

## 3.1 Libraries Used

| Library | Purpose | Notes |
| --- | --- | --- |
| Arduino.h | Core Arduino functions | Built-in |
| WiFi.h | WiFi connectivity | Built-in with ESP32 |
| ModbusIP_ESP8266.h | Modbus TCP server | Works on ESP32 despite name |
| ESP32Servo.h | Servo motor control | ESP32-compatible version |

## 3.2 Modbus Register Map

| Register | Address | Direction | Purpose |
| --- | --- | --- | --- |
| Coil 0 | %QX100.4 | OpenPLC → ESP32 | System enable ON/OFF |
| Discrete Input 0 | %IX100.4 | ESP32 → OpenPLC | Gate status (up/down) |
| Discrete Input 1 | %IX100.5 | ESP32 → OpenPLC | Train approaching flag |
| Discrete Input 2 | %IX100.6 | ESP32 → OpenPLC | Train cleared flag |

## 3.3 State Machine

The crossing gate uses a three-state state machine to manage the gate and LED behavior. This ensures sensors don't interfere with each other and the gate always follows a predictable sequence.

| State | Gate | LED | Watching |
| --- | --- | --- | --- |
| IDLE | UP (open) | OFF | Sensor 1 only |
| GATE_CLOSED | DOWN (blocking) | BLINKING | Sensor 2 only |
| GATE_OPENING | UP (train passing) | BLINKING | Timer (1 second) |

## 3.4 Firmware Iterations

The firmware went through multiple iterations during the session based on testing feedback:

| Version | Change | Reason |
| --- | --- | --- |
| v1 | Initial firmware with basic state machine | First implementation |
| v2 | Replaced delay(1000) with millis() timer in GATE_OPENING | delay() blocked Modbus and WiFi during gate movement |
| v3 | Changed default gate from UP to DOWN | User request — gate should block by default |
| v4 | Changed gate opening timer from 2 seconds to 1 second | User request — faster gate response |
| v5 (final) | Changed default gate back to UP, system OFF also goes to UP | User request — gate open is the safe/normal state |

## 3.5 Network Configuration

| Parameter | Value |
| --- | --- |
| WiFi Network | SCT24 |
| Static IP | 10.13.4.116 |
| Gateway | 10.13.4.1 |
| Subnet | 255.255.255.0 |
| Modbus TCP Port | 502 |

# 4. OpenPLC Configuration

## 4.1 Slave Device Settings

| Field | Value |
| --- | --- |
| Device Name | CrossingGate1 |
| Device Type | Generic Modbus TCP Device |
| Slave ID | 1 |
| IP Address | 10.13.4.116 |
| IP Port | 502 |
| Discrete Inputs Start | 0 |
| Discrete Inputs Size | 3 |
| Coils Start | 0 |
| Coils Size | 1 |
| Input Registers Size | 0 |
| Holding Registers Size | 0 |

## 4.2 Ladder Diagram Variables

| Name | Class | Type | Location | Purpose |
| --- | --- | --- | --- | --- |
| GATE1_BTN | Local | BOOL | %QX1.0 | Manual enable toggle (monitoring page) |
| GATE1_ENABLE | Local | BOOL | %QX100.4 | Command to ESP32 — activates system |

## 4.3 Ladder Rung

One rung controls the crossing gate system:

Contact GATE1_BTN (%QX1.0) → Coil GATE1_ENABLE (%QX100.4)

When GATE1_BTN is forced TRUE from the monitoring page, OpenPLC sends TRUE to the ESP32 Coil 0, activating the crossing gate system. When FALSE, the system shuts down and the gate returns to UP position.

## 4.4 Address Space Note

The %QX0.x address space (0.0 through 0.7) was fully used by previous devices. GATE1_BTN uses %QX1.0, starting the second byte of local output addresses. OpenPLC Editor only supports 0-7 within each byte, so %QX0.8 is invalid — the next available address after %QX0.7 is %QX1.0.

**5. Issues & Changes**

## Issue 1: LED On Constantly at Startup

| Aspect | Details |
| --- | --- |
| Problem | The red LED was permanently on as soon as the ESP32 powered up, before any train was detected. |
| Root Cause | The obstacle IR sensor was either detecting something in its field of view (surface, wall, or the sensitivity potentiometer was set too high), or the sensor outputs HIGH when detecting (opposite of assumed logic). Since the code treated LOW as detected, a consistently HIGH output would not trigger the gate — but if the sensor was outputting LOW constantly (over-sensitive), the LED would stay on. |
| Solution | The obstacle IR sensor has a small blue potentiometer on the back. Adjusting it with a screwdriver changes the detection sensitivity/distance. Turning it calibrates the sensor so the LED on the sensor module turns off when nothing is directly in front of it. |
| Status | RESOLVED |

## Issue 2: WiFi Address Space Full (%QX0.8 Invalid)

| Aspect | Details |
| --- | --- |
| Problem | OpenPLC Editor showed 'Location is invalid' error when trying to assign %QX0.8 to GATE1_BTN. |
| Root Cause | OpenPLC Editor version in use only supports addresses 0 through 7 within each byte of the %QX address space. All 8 slots (%QX0.0 through %QX0.7) were already used by previous devices (SWITCH_BTN, CONFIRM_LED, RADAR_ON_BTN, RADAR_OFF_BTN, RADAR_ENABLE, Bluetrack_WitchButton, Bluetrack_CONFIRM, Track_PowerButtonn). |
| Solution | Changed GATE1_BTN to %QX1.0 and GATE2_BTN to %QX1.1, starting the second byte of the local output address space. |
| Status | RESOLVED |

## Change 1: Gate Default Position

| Aspect | Details |
| --- | --- |
| Problem | Gate default was UP at startup. User wanted gate DOWN by default so the crossing is blocked when the system is not active. |
| Root Cause | Original design had gate UP as the safe/default position. User requirement changed to gate DOWN as the default blocking position. |
| Solution | Changed gateServo.write(GATE_UP) to gateServo.write(GATE_DOWN) in both setup() and the system OFF section of loop(). Later reversed back to GATE_UP after further testing confirmed UP is the correct default. |
| Status | CHANGED |

## Change 2: Gate Opening Timer

| Aspect | Details |
| --- | --- |
| Problem | Gate was staying UP for 2 seconds after train cleared before coming back down. User wanted faster response. |
| Root Cause | Original timer was set to 2000ms (2 seconds) to give the servo enough time to fully reach UP position and allow the train to fully clear. |
| Solution | Changed the timer from 2000ms to 1000ms in the GATE_OPENING state machine case. |
| Status | RESOLVED |

# 6. Final Behavior

- Power on → gate UP, LED OFF, waiting for OpenPLC enable command

- OpenPLC sends FALSE (system OFF) → gate UP, LED OFF

- OpenPLC sends TRUE (system ON) → system activates, watches Sensor 1

- Sensor 1 detects train approaching → LED starts blinking, gate goes DOWN

- Sensor 2 detects train cleared → gate goes UP

- After 1 second → LED stops blinking, gate stays UP, back to IDLE

- System ready for next train — Sensor 1 is watched again

# 7. Successes

| # | Milestone | Status |
| --- | --- | --- |
| 1 | ESP32 #1 wired correctly (IR sensors, RGB LED, servo) | SUCCESS |
| 2 | Firmware compiled and uploaded successfully | SUCCESS |
| 3 | WiFi connected with static IP 10.13.4.116 | SUCCESS |
| 4 | Modbus TCP slave registered on port 502 | SUCCESS |
| 5 | OpenPLC slave device CrossingGate1 configured | SUCCESS |
| 6 | Ladder diagram rung added — GATE1_BTN → GATE1_ENABLE | SUCCESS |
| 7 | State machine working — gate responds to both IR sensors | SUCCESS |
| 8 | LED blinks correctly when train detected, stops when cleared | SUCCESS |
| 9 | Non-blocking timing using millis() — Modbus stays responsive | SUCCESS |
| 10 | System resets cleanly after each train pass | SUCCESS |

# 8. Next Steps

- Build and test ESP32 #2 — Line Tracking IR sensor crossing gate

- Add CrossingGate2 slave device to OpenPLC

- Add GATE2_BTN and GATE2_ENABLE variables to ladder diagram

- Test both crossing gates simultaneously on the LEGO track

- Calibrate servo angles (GATE_UP/GATE_DOWN) to match physical gate position

- Calibrate IR sensor sensitivity using the blue potentiometer on each module
