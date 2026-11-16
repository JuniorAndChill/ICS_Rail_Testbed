**DIN RAIL POWER DISTRIBUTION**

Assembly, Wiring, and Safety Guide

ICS Rail Testbed — IEEE Std 315 / NEC Compliant

Version 1.0 | 25 March 2026

# 1. Safety Requirements

**WARNING: This guide involves AC mains voltage (120VAC/240VAC). Electric shock from mains voltage can cause serious injury or death. All work must be performed by qualified personnel.**

**WARNING: De-energize ALL circuits before making ANY wiring changes. Verify with a non-contact voltage tester (NCVT).**

**WARNING: All AC mains connections must be made through a GFCI-protected outlet per NEC Article 210.8.**

## 1.1 Required Personal Protective Equipment (PPE)

Safety glasses (ANSI Z87.1 rated) — Required when cutting, stripping, or crimping wire.

Insulated tools — Use tools rated to 1000V per IEC 60900 when working near energized conductors.

ESD wrist strap — Required when handling Raspberry Pi, sensors, and electronic components.

## 1.2 Required Tools

| Tool | Purpose |
| --- | --- |
| Non-contact voltage tester (NCVT) | Verify circuits are de-energized before work |
| Digital multimeter (DMM) | Measure DC output voltage and continuity |
| Ferrule crimping tool (AWG 23-7) | Terminate stranded wire with bootlace ferrules |
| Ratchet crimping tool (AWG 22-10) | Crimp ring and spade terminals |
| Wire stripper (18-10 AWG) | Strip insulation from conductors |
| Flat-head screwdriver (3mm, 5mm) | Terminal block screw terminals |
| Phillips-head screwdriver (#2) | DIN rail mounting screws |
| Heat gun or lighter | Activate heat-shrink tubing |
| Wire label printer or markers | Conductor identification per IEEE Std 315 |

# 2. Component Identification

## 2.1 Power Supply Units

| Designator | Model | Input | Output | Mount | Qty |
| --- | --- | --- | --- | --- | --- |
| PS1, PS2 | AC/DC DIN Rail 24V 120W | 100-240VAC | 24VDC/5A | DIN Rail | 2 |
| PS3, PS4 | EDR-120-12 DIN Rail | 100-240VAC | 12VDC/10A | DIN Rail | 2 |
| PS5, PS6 | 72W Variable Bench | 100-240VAC | 3-24VDC/3A | Desktop | 2 |

## 2.2 Terminal Designations (IEEE Std 315)

| Terminal | Color Code | Voltage | Connected Loads |
| --- | --- | --- | --- |
| TB1 (+24V) | Red | +24VDC | PLC inputs, proximity sensors |
| TB2 (24V COM) | Blue | 0V (24V return) | Common for 24V loads |
| TB3 (+12V) | Yellow | +12VDC | Signal LEDs, IR sensors, DCC |
| TB4 (12V COM) | Blue | 0V (12V return) | Common for 12V loads |
| TB5 (+5V) | Orange | +5VDC | RPi USB, servo logic |
| TB6 (PE) | Green/Yellow | Protective Earth | All chassis, DIN rail |

# 3. Assembly Procedure

## 3.1 DIN Rail Mounting

**Step 1:** Select mounting surface (plywood project board or equipment enclosure). Ensure surface is grounded if metallic.

**Step 2:** Position the first 12-inch aluminum DIN rail horizontally. Mark mounting holes at 3-inch intervals using a center punch.

**Step 3:** Drill pilot holes (3.5mm) at each mark. Secure DIN rail with M4 x 12mm pan-head screws and flat washers.

**Step 4:** Mount a second DIN rail 4 inches below the first (for terminal blocks). Verify rails are parallel using a straight edge.

**Step 5:** Test-fit all components before any wiring. Left-to-right order: PS1 (24V) → PS3 (12V) → Terminal Blocks → Fuse holders.

*NOTE: Leave minimum 1 inch between power supplies for heat dissipation per manufacturer specifications.*

## 3.2 AC Mains Wiring (DANGER — 120VAC)

**WARNING: ENSURE THE AC POWER CORD IS UNPLUGGED BEFORE PROCEEDING.**

**WARNING: Verify with NCVT that no voltage is present on any terminal.**

**Step 1:** Cut a 3-conductor 18 AWG power cord (with NEMA 5-15P plug) to appropriate length. The plug end connects to the GFCI outlet.

**Step 2:** Strip the outer jacket back 3 inches. Strip each individual conductor (Black/Line, White/Neutral, Green/Ground) back 8mm.

**Step 3:** Crimp a bootlace ferrule onto each stripped conductor end using the ferrule crimping tool. Tug-test each ferrule.

**Step 4:** Connect Black (Line) wire to PSU terminal marked "L" or "Line". Torque screw terminal to 0.5 Nm.

**Step 5:** Connect White (Neutral) wire to PSU terminal marked "N" or "Neutral". Torque screw terminal to 0.5 Nm.

**Step 6:** Connect Green (Ground) wire to PSU terminal marked "PE" or earth symbol (⏚). Also bond to DIN rail.

**Step 7:** Repeat Steps 4-6 for the second power supply (PS3/12V), using a separate cord or splitting from a fused distribution block.

*NOTE: Use a 5A fuse inline with each PSU AC input for additional protection. Mount fuse holders on the DIN rail.*

## 3.3 DC Output Wiring

**Step 1:** From PS1 (+24V output), run red 18 AWG wire to Terminal Block TB1 (+24V bus). Crimp ferrule, insert, torque.

**Step 2:** From PS1 (-V/COM output), run blue 18 AWG wire to Terminal Block TB2 (24V COM bus).

**Step 3:** From PS3 (+12V output), run yellow 18 AWG wire to Terminal Block TB3 (+12V bus).

**Step 4:** From PS3 (-V/COM output), run blue 18 AWG wire to Terminal Block TB4 (12V COM bus).

**Step 5:** Connect a green/yellow wire from each PSU PE terminal to TB6 (Protective Earth bus) and to the DIN rail.

**Step 6:** From TB1/TB2, run paired wires to each 24V load: proximity sensors (NPN, 3-wire), PLC analog inputs.

**Step 7:** From TB3/TB4, run paired wires to each 12V load: crossing signal LEDs, IR sensor modules, DCC decoder power input.

**Step 8:** Apply heat-shrink tubing to all exposed crimp connections. Activate with heat gun at 120°C.

## 3.4 Verification and Commissioning

**Step 1:** Visual inspection: verify all connections are tight, no exposed copper, ferrules seated, heat-shrink applied.

**Step 2:** Continuity test: use DMM continuity mode to verify each wire from source terminal to destination terminal.

**Step 3:** Insulation test: verify no continuity between +V and COM, between +V and PE, or between different voltage rails.

**Step 4:** Energize: plug in the GFCI cord. Verify GFCI trips correctly by pressing the TEST button. Reset.

**Step 5:** Measure DC outputs under no-load: TB1 should read 24.0V ±0.5V; TB3 should read 12.0V ±0.3V.

**Step 6:** Connect loads one at a time, measuring voltage at each terminal to detect sag or wiring errors.

**Step 7:** Record all measurements in the commissioning log. Photograph the completed assembly for documentation.
