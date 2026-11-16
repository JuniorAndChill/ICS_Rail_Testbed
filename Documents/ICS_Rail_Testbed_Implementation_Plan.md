**ICS RAIL TESTBED**

Implementation Plan

Cyber-Physical Threat Modeling of IT/OT Lateral Movement
in Railroad Control Systems

University of the Incarnate Word

Senior Design Project — OPS-Warfare Division

Document Version: 1.0  |  Date: 25 March 2026  |  Classification: Educational / Lab Use

# 1. Executive Summary

ICS Rail Testbed establishes a Purdue Model-compliant IT/OT research laboratory simulating railroad control system infrastructure. The lab uses HO-scale model railroad hardware integrated with industrial-grade DIN rail power distribution, OpenPLC-based programmable logic controllers on Raspberry Pi 4 platforms, and a full network security monitoring stack including Zeek IDS, Pi-hole DNS sinkhole, and centralized syslog. The project delivers 10 documented adversarial attack scenarios against Modbus/TCP and ICS protocols, with full PCAP evidence, Zeek correlation logs, and IEEE-formatted analysis. All physical builds use DIN rail mounted components with properly terminated wiring per NEC and IEEE standards.

# 2. Consolidated Bill of Materials (Approved Parts List)

The following parts list is the authoritative procurement document. Total budget: $1,381.19 (including tax).

| Item | Qty | Vendor | Cost | Purpose |
| --- | --- | --- | --- | --- |
| Rail Chief Train Model Set | 1 | Bachmann | $365.00 | Locomotive, cars, track, power pack |
| 52' Flat Car - Union Pacific #59486 | 1 | Bachmann | $35.00 | Rolling stock for cargo simulation |
| 52' Flatcar - Desert Camouflage w/ Crates | 2 | Bachmann | $70.00 | Freight cargo simulation |
| Steel Alloy Expander Set (HO) | 1 | Bachmann | $120.00 | Track expansion |
| Remote Turnout - Right (HO) | 1 | Bachmann | $51.75 | Switch simulation (servo-actuated) |
| Remote Turnout - Left (HO) | 1 | Bachmann | $51.75 | Switch simulation (servo-actuated) |
| DCC Control Box w/ Turnout Decoder | 2 | Bachmann | $103.50 | Digital command control |
| IR Infrared Sensor Module (6pcs) | 1 | Amazon | $6.99 | Track occupancy detection |
| Inductive Proximity Sensor M12 (4 pack) | 1 | Amazon | $13.99 | Metal detection for trains |
| SG90 Micro Servo Motor (5 pcs) | 1 | Amazon | $7.99 | Gate/switch actuation via OpenPLC |
| PLA Matte Filament 1.75mm (1KG) | 2 | Amazon | $49.18 | 3D printed enclosures/mounts |
| 18 AWG Wire 5-Roll (1000ft total) | 1 | Amazon | $49.99 | Signal and power wiring |
| PCB Board Kit (32 pcs, 5 sizes) | 1 | Amazon | $7.99 | Sensor interface circuits |
| Breadboard + Jumper Wire Kit | 1 | Amazon | $9.99 | Prototyping |
| 74HCxx/74LSxx Logic IC Kit (49 pcs) | 1 | Amazon | $11.99 | Signal conditioning |
| ELEGOO Electronics Fun Kit | 1 | Amazon | $15.99 | Power supply module, components |
| NETGEAR GS308 Unmanaged Switch | 1 | NETGEAR | $18.99 | OT network (VLAN 20) |
| NETGEAR GS308E Managed Switch | 1 | NETGEAR | $27.99 | IT/OT VLAN segmentation (802.1Q) |
| 72W Variable Power Supply (3-24V) | 2 | Amazon | $31.98 | Bench testing and sensor power |
| EDR-120-12 DIN Rail PSU (12V 120W) | 2 | Amazon | $55.98 | DIN rail 12VDC for signals/sensors |
| AC/DC DIN Rail Supply (24V 120W) | 2 | Amazon | $67.12 | DIN rail 24VDC for PLC I/O |
| Terminal Block Strip Kit (12-pos) | 1 | Amazon | $13.89 | Power distribution |
| DIN Rail Terminal Block Kit | 1 | Amazon | $19.99 | Signal wiring distribution |
| Aluminum DIN Rail 12" (10 pcs) | 1 | Amazon | $15.99 | Component mounting |
| Ratchet Crimping Tool (AWG 22-10) | 1 | Amazon | $18.67 | Wire termination |
| Heat Shrink Connector Kit (320 pcs) | 1 | Amazon | $19.99 | Waterproof crimp connectors |
| Ferrule Crimper + Connector Kit (1800) | 1 | Amazon | $14.24 | DIN rail wire ferrules |

# 3. Implementation Phases

## Phase 1: Physical Build & DIN Rail Assembly (Week 1-2)

| # | Task Description |
| --- | --- |
| 1 | Unbox and inventory all parts against BOM |
| 2 | Mount DIN rails on project board/enclosure |
| 3 | Install and wire 24V and 12V DIN rail power supplies |
| 4 | Install terminal blocks and wire power distribution bus |
| 5 | Crimp ferrules on all conductor ends per NEC standards |
| 6 | Wire IR sensors and proximity sensors to PCB interface boards |
| 7 | Wire SG90 servos to OpenPLC Raspberry Pi GPIO |
| 8 | Assemble HO scale track layout with turnouts |
| 9 | Connect DCC decoder boxes to turnout motors |
| 10 | Test all power rails with multimeter (verify 12V, 24V, 5V) |
| 11 | Label all wires per IEEE Std 315 conventions |
| 12 | Document wiring with photographs and schematic updates |

## Phase 2: IT/OT Network Infrastructure (Week 2-3)

| # | Task Description |
| --- | --- |
| 1 | Configure NETGEAR GS308E managed switch (VLAN 10 IT, VLAN 20 OT, SPAN port) |
| 2 | Flash Raspberry Pi images using Packer build pipeline |
| 3 | Run bootstrap scripts on all four nodes (OpenPLC, Zeek, Pi-hole, Log Server) |
| 4 | Assign static IPs per SOP architecture diagram |
| 5 | Configure cross-VLAN ACLs (SSH to OT, rsyslog to IT) |
| 6 | Validate end-to-end connectivity with validate-lab.sh |
| 7 | Deploy Pi-hole DNS sinkhole on VLAN 10 |
| 8 | Configure Zeek IDS sensor with SPAN mirror of VLAN 20 |
| 9 | Verify Modbus/TCP protocol parsing on Zeek |
| 10 | Test rsyslog forwarding from OT to centralized log server |
| 11 | Document network topology and IP assignments |

## Phase 3: OpenPLC Programming & Sensor Integration (Week 3-4)

| # | Task Description |
| --- | --- |
| 1 | Install OpenPLC runtime on Raspberry Pi 4 |
| 2 | Load PSM driver (openplc_psm_driver.py) and verify GPIO mapping |
| 3 | Develop ladder logic for track occupancy detection |
| 4 | Develop ladder logic for signal state management (Red/Green) |
| 5 | Develop ladder logic for turnout/switch servo control |
| 6 | Develop ladder logic for temperature monitoring safety interlock |
| 7 | Create Function Block Diagrams (FBD) for all control sequences |
| 8 | Test sensor-to-PLC data flow (%IW0, %IX0.0, %QX0.0) |
| 9 | Validate Modbus register mapping via pymodbus read tests |
| 10 | Build dashboard web interface for real-time sensor state display |
| 11 | Document all ladder logic with IEEE Std 61131-3 annotations |

## Phase 4: CVE Research & Attack Development (Week 4-6)

| # | Task Description |
| --- | --- |
| 1 | Research and document 10 CVEs relevant to rail/ICS infrastructure |
| 2 | Develop attack scripts for all 10 scenarios |
| 3 | Build analysis criteria matrix (detection, prevention, impact) |
| 4 | Create Zeek signature rules for each attack vector |
| 5 | Execute Attack 1: Ghost Train Signaling Spoof |
| 6 | Execute Attack 2: Modbus Coil Injection (existing modbus_inject.py) |
| 7 | Execute Attack 3: PLC Logic Wipe / Denial of Service |
| 8 | Execute Attack 4: Session Hijack via Web Portal |
| 9 | Execute Attack 5: Default Credential Exploitation |
| 10 | Execute Attacks 6-10 (RF jamming sim, ransomware sim, backup module, power cycle, lateral pivot) |
| 11 | Capture PCAP evidence for each attack |
| 12 | Record Zeek logs and correlate with attack timeline |
| 13 | Document all findings per IEEE format |

## Phase 5: Analysis, Dashboard & Documentation (Week 6-7)

| # | Task Description |
| --- | --- |
| 1 | Analyze attack results against defense criteria matrix |
| 2 | Build comprehensive sensor dashboard with live Modbus data |
| 3 | Create wire diagrams and schematics for all subsystems |
| 4 | Compile SOP binder with summarized guides (TI/TO format) |
| 5 | Write IEEE-formatted research paper sections |
| 6 | Prepare OTICS conference abstract and poster |
| 7 | Peer review all documentation within team |
| 8 | Final integration test of complete lab environment |

## Phase 6: Presentation & Deliverables (Week 7-8)

| # | Task Description |
| --- | --- |
| 1 | Prepare demonstration environment |
| 2 | Build split-screen demonstration (IT view vs OT reality) |
| 3 | Rehearse attack scenario walkthroughs |
| 4 | Compile final SOP binder |
| 5 | Submit all deliverables to Dr. De La Torre Parra |
| 6 | Present at OTICS conference |

# 4. Network Architecture

VLAN 10 — IT Management (192.168.10.0/24): Central Log Server (.100), Pi-hole DNS (.101), Zeek IDS (.102), Admin Workstations (.x). VLAN 20 — OT Control (192.168.20.0/24): OpenPLC RPi4 (.101), ScadaBR HMI (.102). SPAN port mirrors all VLAN 20 traffic to Zeek sensor eth1 (no IP, read-only capture). Cross-VLAN ACL permits only SSH (TCP 22) from IT to OT and rsyslog (TCP 514) from OT to IT.

# 5. DIN Rail Power Distribution

## 5.1 Safety Precautions

**WARNING: AC mains wiring must comply with NEC Article 110 and local building codes. All AC connections must be made with the circuit de-energized. Use a GFCI-protected outlet for all lab AC power. Verify zero voltage with a non-contact voltage tester before working on any AC wiring.**

## 5.2 Power Supply Specifications

| Parameter | 24V PSU (x2) | 12V PSU (x2) | 72W Bench (x2) |
| --- | --- | --- | --- |
| Model | AC/DC DIN Rail 24V 120W | EDR-120-12 | Minidodoca 72W Variable |
| Input | 100-240VAC 50/60Hz | 100-240VAC 50/60Hz | 100-240VAC 50/60Hz |
| Output | 24VDC @ 5A | 12VDC @ 10A | 3-24VDC @ 3A |
| Mounting | 35mm DIN Rail | 35mm DIN Rail | Benchtop (cable) |
| Purpose | PLC I/O, proximity sensors | Signal lights, IR sensors | Testing, servo development |

## 5.3 Wiring Procedure

1. Mount DIN rails horizontally on the project board using M4 screws at 12-inch intervals.

2. Snap 24V and 12V power supplies onto the DIN rail with the AC input terminals facing left.

3. Install terminal blocks on the DIN rail adjacent to each power supply for distribution.

4. Cut 18 AWG wire to length for AC input leads. Strip 8mm from each end.

5. Crimp bootlace ferrules onto all stripped wire ends using the ferrule crimping tool.

6. Connect AC Line (black) and Neutral (white) from the GFCI cord to the PSU L and N terminals.

7. Connect the ground wire (green) to the PSU earth/PE terminal.

8. Connect PSU +V and -V (or COM) outputs to the distribution terminal blocks.

9. Run 18 AWG leads from terminal blocks to each sensor, actuator, and controller input.

10. Verify all connections with a multimeter: measure output voltage under no-load, then under load.

11. Label all conductors using wire labels per IEEE Std 315-1975 device designations.

12. Apply heat shrink tubing to all exposed crimp connections.

# 6. Deliverables Matrix

| # | Deliverable | Format | Phase |
| --- | --- | --- | --- |
| 1 | Physical Build Wiring Schematics | PDF/Visio (IEEE Std 315) | Phase 1 |
| 2 | DIN Rail Power Distribution Diagram | PDF Schematic | Phase 1 |
| 3 | Network Architecture Diagram | PDF/Visio | Phase 2 |
| 4 | VLAN Configuration Documentation | Markdown + Config Files | Phase 2 |
| 5 | IT Infrastructure Automation Scripts | Bash (tested, validated) | Phase 2 |
| 6 | Ladder Logic Programs (all sequences) | OpenPLC .st files | Phase 3 |
| 7 | Function Block Diagrams | PDF per IEEE 61131-3 | Phase 3 |
| 8 | CVE Research Report (10 vulnerabilities) | IEEE-formatted DOCX | Phase 4 |
| 9 | Attack Scripts (10 scenarios) | Python + documentation | Phase 4 |
| 10 | Attack Analysis Matrix | XLSX + DOCX | Phase 4 |
| 11 | PCAP Evidence Archive | PCAP + Zeek logs | Phase 4 |
| 12 | Sensor Dashboard Web Application | HTML/JS/Python | Phase 5 |
| 13 | Comprehensive SOP Binder (TI/TO) | DOCX | Phase 5 |
| 14 | IEEE Research Paper | IEEE Conference Format | Phase 5 |
| 15 | OTICS Conference Abstract & Poster | PDF | Phase 6 |
