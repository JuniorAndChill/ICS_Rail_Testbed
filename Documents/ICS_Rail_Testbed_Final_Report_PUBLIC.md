**ICS Rail Testbed:**

**Cyber-Physical Threat Modeling of IT/OT**

**Lateral Movement in Railroad Systems Testbed**

By

**Team DJ:**

Daniel Critchlow Jr.

Joseph A. Montez Jr.

**ENGR 4380 Senior Design 2 Final Project Report**

**UNIVERSITY OF THE INCARNATE WORD**

**School of Mathematics, Science, and Engineering**

**Department of Engineering, Computing, and Cybersecurity (ECCS)**

**May 2026**

# Acknowledgments

The Senior Design project was supported by the University of the Incarnate Word (UIW) and would not have been possible without the contributions of several UIW faculty members and industry partners. We would like to thank the special contributions made by:

- Dr. Okan Caglayan — Faculty Lead, Department of Engineering

- Dr. Gonzalo De la Torre Parra — Instructor/Professor, CSEC

- Dr. Marta Del Rio — CIS Faculty

- Professor Ammari — CIS/CSEC Faculty

- Professor Idar — CIS Faculty

- CNF Technologies (Nathan Vasquez) — Industry Partner and Sponsor

- Industry and Lab Partners

- Additional lab equipment and fabrication resources

- Luis F. De Oliveira, Oscar S. Hawit Sabillon, Nathan Ortiz — Contributing students

# Table of Contents

Acknowledgments......................................................................................................................2

List of Figures...........................................................................................................................5

List of Tables............................................................................................................................6

Abstract.....................................................................................................................................7

1. Introduction..........................................................................................................................8

1.1 Motivation/Problem.......................................................................................................8

1.2 Research Objectives......................................................................................................8

1.3 Project Requirements and Specifications.....................................................................9

1.4 Organization.................................................................................................................9

2. System Design....................................................................................................................10

2.1 Design Overview..........................................................................................................10

2.2 Design Standards.........................................................................................................12

2.3 Prototype Design.........................................................................................................13

2.4 Prototype Enclosure (Construction).............................................................................16

2.5 Prototype Testing........................................................................................................18

3. Results.................................................................................................................................19

3.1 Field Testing Analysis.................................................................................................19

3.2 Problems Encountered/Analysis of Problems Solved..................................................22

3.3 Timeline......................................................................................................................23

3.4 Recommendations.......................................................................................................24

4. Conclusion..........................................................................................................................25

4.1 Summary of Project.....................................................................................................25

4.2 Lessons Learned..........................................................................................................25

4.3 Future Work................................................................................................................26

References...............................................................................................................................27

Appendix A: Attack Scripts and Code.....................................................................................28

Appendix B: PLC I/O Variable Map and Datasheets..............................................................32

# List of Figures

Figure 1: Purdue Model for OT Cybersecurity Architecture.................................................10

Figure 2: Environment Inventory and Network Topology.....................................................13

Figure 3: ESP-32 Slave Device Pin Configuration................................................................14

Figure 4: Allen-Bradley MicroLogix 1100 Series B PLC with DIN Rail Power Supply......15

Figure 5: PowerTrack ESP32 Slave Device Configuration...................................................15

Figure 6: SwitchTrack ESP32 Slave Device Configuration..................................................16

Figure 7: Testbed #1 — Bricks in a Loop LEGO Layout..................................................17

Figure 8: Testbed #2 — Bachmann EZ-Track HO Scale Layout......................................17

Figure 9: 3D Printed Models for Testbed Infrastructure.......................................................17

Figure 10: OpenPLC Dashboard and Monitoring Interface....................................................18

Figure 11: IronSpider Web Attack — Railroad Control System Dashboard......................21

Figure 12: Physical Switch Track Manipulation Demonstration...........................................22

# List of Tables

Table 1: Cost Comparison — Existing ICS/SCADA Training Solutions.........................11

Table 2: Environment Inventory............................................................................................13

Table 3: PLC I/O Variable Map.............................................................................................14

Table 4: CVE Research — Relevant Vulnerabilities for Railroad ICS..............................19

Table 5: Attack Criteria Framework Summary......................................................................20

Table 6: CVSS Score Estimation............................................................................................22

Table 7: Mitigation Recommendations..................................................................................23

Table 8: Bill of Materials.......................................................................................................16

Table 9: Project Timeline.......................................................................................................24

# Abstract

The convergence of information technology (IT) and operational technology (OT) has significantly expanded the cyber-physical attack surface of rail transportation infrastructure. The Cybersecurity and Infrastructure Security Agency (CISA) reported over 1,200 Industrial Control Systems (ICS) vulnerabilities in 2024 alone, with the transportation sector rated at critical risk level for both likelihood and impact. ICS Rail Testbed addresses this threat by designing, building, and validating a research-grade cyber-physical training platform that demonstrates lateral movement of cyber threats from IT protocols into safety-critical OT systems, specifically targeting railroad signaling and switching infrastructure.

The testbed integrates two complementary physical platforms: a custom LEGO-based microcontroller-controlled train set (Testbed #1) and a Bachmann EZ-Track HO Scale industrial-grade layout (Testbed #2), both governed by OpenPLC Runtime on Raspberry Pi 4B as the Modbus TCP master controller with ESP32 slave devices managing track switching, power distribution, and radar sensing. The system architecture follows the Purdue Model for OT cybersecurity with devices deployed across Levels 0 through 3. A Raspberry Pi running Kali Linux serves as the dedicated attacker node.

Ten adversarial scenarios were designed, documented, and validated against documented Common Vulnerabilities and Exposures (CVEs) including CVE-2022-1161, CVE-2023-3595, and CVE-2025-1727. Attack execution achieved a 9.5 out of 10 success rate, with detection latency under 50 milliseconds. Physical consequences were confirmed, including unauthorized track switch movement, sensor system disablement, and complete device denial of service via Modbus TCP connection flooding. The project demonstrates that the Modbus TCP protocol’s lack of authentication enables any network-connected device to achieve full read/write control over all slave devices and the master PLC using only standard tools available in Kali Linux.

The platform achieves 60–75% cost savings compared to full-scale hardware alternatives, operating within a $1,500 budget ceiling at a final cost of $1,355.07. All documentation follows IEEE standards, and the system aligns with CISA ICS-CERT advisories, NIST SP 800-82, and IEC 62443 guidelines. The project establishes a foundation for future integration with machine learning-based anomaly detection and expansion to wireless communication attack vectors.

# 1. Introduction

## 1.1 Motivation/Problem

Industrial Control Systems (ICS) and Supervisory Control and Data Acquisition (SCADA) networks are increasingly targeted by cyber adversaries seeking to disrupt critical infrastructure. Rail transportation systems rely on Programmable Logic Controllers (PLCs) to manage signaling, switching, and train control operations. The IT/OT convergence trend has expanded the attack surface, enabling standard IT protocols to facilitate lateral movement into safety-critical OT systems. Physical consequences of successful attacks include train derailment, head-on collision, signal manipulation, and complete transportation shutdown.

Real-world railroad cyber incidents underscore the severity of this threat. In 2016, the San Francisco Municipal Railway (Muni) suffered a ransomware attack resulting in over $50,000 in losses. In 2020, European Rail SCADA systems experienced safety exposure through unauthorized access. Belarus Rail was disrupted in 2022 through logistics system compromise. In 2023, the Polish rail system was attacked using radio stop commands, halting over 20 trains. Most recently in 2026, German railway booking systems were targeted by distributed denial-of-service (DDoS) attacks.

Educational institutions lack accessible, hands-on environments for studying these IT/OT lateral movement threats. Existing training solutions are either prohibitively expensive (full-scale rail testbeds at $500K–$2M+), lack physical impact demonstration (software-only simulations at $10K–$50K), or are not rail-specific (generic ICS training kits at $15K–$100K). ICS Rail Testbed bridges this gap by providing a cost-effective, research-grade cyber-physical training platform using commodity hardware.

## 1.2 Research Objectives

- Design and deploy a Purdue Model-compliant OT research environment with network isolation between IT and OT planes.

- Construct a physical rail infrastructure model with microcontroller-controlled trains and industrial-grade track switching for demonstrating ICS attack scenarios.

- Research, design, and execute 10 distinct adversarial scenarios ranging from network reconnaissance to coordinated multi-vector attacks with physical impact.

- Implement real-time sensor state monitoring through a web dashboard with Modbus TCP polling.

- Develop a Safety Watchdog capability targeting sub-100ms detection of unauthorized Modbus commands.

- Document all procedures, attack scripts, and analysis in accordance with IEEE standards for reproducibility and knowledge transfer.

- Achieve 60–75% cost savings versus full-scale hardware alternatives within a $1,500 budget ceiling.

## 1.3 Project Requirements and Specifications

The system employs OpenPLC Runtime on Raspberry Pi 4B as the Modbus TCP master controller communicating with ESP32 slave devices over TCP port 502. The physical demonstration includes working train sets with microcontroller-controlled actuation capable of demonstrating collision and derailment scenarios. All ESP32 slave devices operate as Modbus TCP slaves with specific functional roles: track switching (servo actuation), power distribution (relay control), and radar sensing (ultrasonic detection). The platform utilizes Allen-Bradley MicroLogix 1100 Series B PLCs for industrial-grade OT representation with 10 digital inputs at 24V DC, 6 relay outputs, EtherNet/IP and Modbus/TCP communications, and RSLogix 500 programming capability. A Raspberry Pi running Kali Linux serves as the dedicated attacker node on the same network segment. The project operates within a $1,500 budget ceiling with 10% contingency reserve.

## 1.4 Organization

This report is organized as follows: Section 2 details the system design including the Purdue Model architecture, design standards, hardware and software specifications, and prototype construction. Section 3 presents results including attack execution outcomes, vulnerability analysis, problems encountered, timeline, and mitigation recommendations. Section 4 provides conclusions encompassing project summary, lessons learned, and future work directions. References follow IEEE publication format, and appendices contain attack scripts and PLC datasheets.

# 2. System Design

## 2.1 Design Overview

ICS Rail Testbed integrates two complementary architectural levels based on the Purdue Model for OT cybersecurity. The system architecture spans four levels: Level 0 (Field Zone) contains sensors, actuators, track switches, and signals; Level 1 (Control Zone) houses the PLCs and RTUs including the Allen-Bradley MicroLogix 1100; Level 2 (Supervisory Zone) contains engineering workstations, HMI, and historians; and Levels 3–4 (Enterprise/IT Zone) represent business systems and the adversary entry point. The lateral movement attack path traverses from the Enterprise/IT Zone downward through the Supervisory and Control Zones into the Field Zone.

***2.1.1 Existing Projects and Designs***

Several commercial and academic ICS testbeds exist for cybersecurity training. Full-scale rail testbeds cost $500K to $2M+ and offer high fidelity but are extremely expensive. Software-only simulations range from $10K to $50K and offer low cost but cannot demonstrate physical impact. Generic ICS training kits cost $15K to $100K and are commercially available but are not rail-specific. The IronSpider research demonstrated web-based PLC malware targeting MicroLogix embedded web servers, where client-side JavaScript manipulates physical actuators through a technician’s browser session. ICS Rail Testbed bridges the gap between these approaches by combining commodity Raspberry Pi hardware with industrial Allen-Bradley PLCs to create a dual-fidelity training environment at approximately $1,500.

| Solution | Cost | Advantages | Limitations |
| --- | --- | --- | --- |
| Full-Scale Rail Testbed | $500K – $2M+ | High fidelity | Extremely expensive |
| Software-Only Simulation | $10K – $50K | Low cost | No physical impact demo |
| Generic ICS Training Kit | $15K – $100K | Available | Not rail-specific |
| ICS Rail Testbed | ~$1,355 | Rail-specific, physical impact | Scale model, not full-scale |

*Table 1: Cost Comparison — Existing ICS/SCADA Training Solutions*

***2.1.2 Relevant Technologies***

- OpenPLC Runtime — IEC 61131-3 compliant PLC programming on Raspberry Pi 4B, serving as the Modbus TCP master controller.

- Allen-Bradley MicroLogix 1100 Series B — Industrial-grade PLC with EtherNet/IP, Modbus/TCP, 10 digital inputs at 24V DC, and 6 relay outputs. Processor model 1763-L16BWA with 8K words user program memory.

- ESP32 Microcontrollers — Wi-Fi and BLE capable slave devices operating as Modbus TCP slaves for track switching (servo actuation), power distribution (relay control), and radar sensing (ultrasonic detection).

- Raspberry Pi (Kali Linux) — Dedicated attacker node with Metasploit framework for Modbus protocol exploitation.

- Bachmann EZ-Track HO Scale — Industrial-grade model railroad track system with remote turnouts, DCC control, and steel alloy expansion capability.

***2.1.3 Market Analysis***

The project achieves significant cost savings by utilizing open-source software (OpenPLC Runtime, Kali Linux, Metasploit) combined with commodity hardware (Raspberry Pi, ESP32) and HO-scale model railroad components. The total project cost of $1,355.07 represents approximately 60–75% savings compared to generic ICS training kits and over 99% savings compared to full-scale rail testbeds. All equipment beyond the $1,500 budget ceiling, including Allen-Bradley MicroLogix 1100 PLCs and the original LEGO train set, was provided by CNF Technologies and partner lab resources.

## 2.2 Design Standards (Safety and Regulations)

The project adheres to the following safety regulations and industry standards, organized by governing body:

**CISA / DHS (Cybersecurity and Infrastructure Security Agency):**

- ICS-CERT Advisories — Rockwell Automation/MicroLogix vulnerability tracking for threat modeling.

- Cross-Sector Cybersecurity Performance Goals (CPGs) — Baseline security measures for critical infrastructure.

- Rail Cybersecurity Guide — Sector-specific guidance for railroad control system protection.

**NIST (National Institute of Standards and Technology):**

- SP 800-82 Rev. 3 — Guide to Operational Technology (OT) Security, informing the Purdue Model architecture.

- SP 800-53 — Security and Privacy Controls for information systems and organizations.

- Cybersecurity Framework (CSF) — Risk management framework for critical infrastructure sectors.

**IEEE / IEC (Industry Standards Organizations):**

- IEC 62443 — Industrial Automation and Control Systems (IACS) security standard series.

- IEEE 1686 — Standard for Intelligent Electronic Device (IED) cybersecurity capabilities.

- Modbus/TCP Standards — Industrial communication protocol specifications.

- IEC 61131-3 — PLC programming languages governing the OpenPLC address space (%IX, %QX, %IW, %QW).

## 2.3 Prototype Design

***2.3.1 Project Hardware and Software Details***

**Network Architecture and Environment Inventory**

The testbed network operates on the 10.13.4.0/24 subnet with all devices communicating via Modbus TCP on port 502. The Raspberry Pi 4B running OpenPLC Runtime serves as the master controller at IP address 10.13.4.109. Four ESP32 slave devices manage specific field-level functions: SwitchTrack (10.13.4.205) controls the outer track switch servo, SwitchTrack 2/BlueTrain (10.13.4.207) controls the inner track switch servo, Radar (10.13.4.208) operates the ultrasonic proximity sensor, and PowerTrack (10.13.4.89) manages track power distribution via relay. The Raspberry Pi running Kali Linux at 10.13.4.185 serves as the dedicated attacker node connected via Ethernet.

| Device | IP Address | Role | Protocol |
| --- | --- | --- | --- |
| Raspberry Pi 4B (OpenPLC) | 10.13.4.109 | Master Controller / Brain | Modbus TCP Master |
| ESP32 - SwitchTrack | 10.13.4.205 | Switch Track Slave 1 | Modbus TCP Slave |
| ESP32 - SwitchTrack 2 (BlueTrain) | 10.13.4.207 | Switch Track Slave 2 | Modbus TCP Slave |
| ESP32 - Radar | 10.13.4.208 | Radar Sensor Slave | Modbus TCP Slave |
| ESP32 - PowerTrack | 10.13.4.89 | Power Track Slave | Modbus TCP Slave |
| Raspberry Pi (Kali Linux) | 10.13.4.185 | Attacker Node | Ethernet (eth0) |

*Table 2: Environment Inventory*

**PLC I/O Variable Map**

The OpenPLC master controller manages the following I/O variables mapped to Modbus addresses for controlling slave device operations:

| Variable Name | Type | Location | Function |
| --- | --- | --- | --- |
| SWITCH_BTN | BOOL | %QX0.0 | Switch button command |
| SWITCH_CMD | BOOL | %QX100.0 | Switch track command to slave |
| IN_POSITION | BOOL | %IX100.0 | Switch in-position feedback |
| CONFIRM_LED | BOOL | %QX0.1 | Position confirmation LED |
| RADAR_ON_BTN | BOOL | %QX0.2 | Radar enable button |
| RADAR_OFF_BTN | BOOL | %QX0.3 | Radar disable button |
| RADAR_ENABLE | BOOL | %QX0.4 | Radar enable state (SET/RESET) |
| RADAR_CMD | BOOL | %QX100.1 | Radar command to slave |
| BlueTrack_WitchButton | BOOL | %QX0.5 | Blue train switch button |
| BlueTrack_SWITCH_CMD0 | BOOL | %QX100.2 | Blue train switch command |
| Track_PowerButtonn | BOOL | %QX0.7 | Track power button |
| Track_Power_CMD | BOOL | %QX100.3 | Track power command |

*Table 3: PLC I/O Variable Map*

**Slave Device Configurations**

PowerTrack (ESP32): This device uses a battery bank for power and energizes the ESP32 microcontroller. The ESP32 connects the circuit with a normally-open (NO) relay which powers the track. An emergency stop button and potentiometer provide two independent methods for stopping track power. The ESP32 communicates wirelessly with the master controller via Modbus TCP.

SwitchTrack (ESP32 x2): Each SwitchTrack device uses a battery bank for power and contains scripts to actuate a 9G micro servo between transpose and straight positions. An LCD display monitors the current switch status and Modbus connection state. Each device is wireless-capable and communicates with the master controller via Modbus TCP.

Radar (ESP32): The radar slave device operates an ultrasonic proximity sensor for train detection. It reports occupancy data back to the master controller for collision avoidance logic.

**Allen-Bradley MicroLogix 1100 Series B PLC**

The project incorporates Allen-Bradley MicroLogix 1100 Series B PLCs as industrial-grade OT components. The processor model 1763-L16BWA provides 16-point I/O with 10 digital inputs at 24V DC and 6 relay outputs. Communication is supported via EtherNet/IP and Modbus/TCP. Programming is performed using RSLogix 500/Micro with 8K words of user program memory. The PLCs are mounted on DIN rails alongside Selec RPS120-24-CU power supplies providing 24VDC at 120W output. These units represent the real-world industrial control systems found throughout railroad signaling infrastructure.

**Dashboard Integration**

Real-time sensor state monitoring is achieved through a web dashboard architecture. The data flow operates as follows: PLC data is transmitted via Modbus/TCP to a polling service, which stores values in a database, which feeds the web dashboard display. Displayed metrics include switch positions (left/right), sensor connectivity status, new device ID detection alerts, and system runtime information. The dashboard provides both a Control Panel view for operator-level interaction and an Advanced Controls view for direct coil manipulation during testing.

## 2.4 Prototype Enclosure (Construction)

***2.4.1 Materials and Cost***

The project operates within a $1,500 budget ceiling funded by CNF Technologies. Additional partner-loaned equipment included two Allen-Bradley MicroLogix 1100 PLCs and the original LEGO train set. All newly procured materials are itemized in the Bill of Materials below.

***2.4.2 Bill of Materials***

| Item | Qty | Total |
| --- | --- | --- |
| Rail Chief Train Set | 1 | $365.00 |
| EZ-Track Steel Alloy Expander Set | 1 | $120.00 |
| Remote Turnouts (Left + Right) | 2 | $103.50 |
| DCC Control w/Turnout Decoder | 2 | $103.50 |
| 52’ Flat Cars (Union Pacific + Desert Camo) | 3 | $105.00 |
| DIN Rail Power Supplies (12V EDR-120 + 24V RPS120) | 4 | $111.96 |
| Terminal Blocks & DIN Rails | 1 | $49.87 |
| IR/Proximity Sensors | 2 | $20.98 |
| Network Switches (Managed + Unmanaged) | 2 | $46.98 |
| Electronics & Wire (breadboard, PCB, wire, IC kit, etc.) | 1 | $175.83 |
| PLA 3D Printer Filament | 2 | $49.18 |
| Subtotal |  | $1,251.80 |
| Sales Tax (8.25%) |  | $103.27 |
| TOTAL |  | $1,355.07 |

*Table 8: Bill of Materials*

Budget utilization stands at 90% of the $1,500 ceiling with 10% contingency reserved.

**Physical Construction**

Testbed #1 (Bricks in a Loop): A custom LEGO train set built specifically for microcontroller integration. The layout features a dual-loop configuration with multiple track switches, 3D-printed buildings, a 3D-printed radar suite with ultrasonic sensor, and ESP32 slave devices controlling all physical elements. Custom 3D-printed DIN rail holders, device mounts, DIN rails, LEGO track switcher mechanisms, and train station houses were fabricated using Bambu Lab 3D printers with PLA matte filament.

Testbed #2 (Bachmann EZ-Track HO Scale): An out-of-the-box Bachmann Rail Chief train set provided by CNF Technologies. This layout provides a more industrial-grade demonstration platform with steel alloy track, remote turnouts, DCC control decoders, and connection to the Allen-Bradley MicroLogix 1100 PLCs via 24VDC controls. The HO-scale layout integrates with the same OpenPLC master controller and network infrastructure as Testbed #1.

## 2.5 Prototype Testing

IDS validation was performed using modbus_inject.py to send Modbus FC5 (Write Single Coil) commands to the OpenPLC node, physically actuating servos and verifying detection in Zeek modbus.log. End-to-end testing validated the full chain from injection through physical actuation, network detection, and dashboard state reflection. The team verified that all slave devices accept Modbus TCP commands from the master controller and confirmed that unauthorized commands from the attacker node bypass all ladder logic safety checks. Physical switch track manipulation was demonstrated via network commands, confirming cyber-physical impact capability.

# 3. Results

## 3.1 Field Testing Analysis

**CVE Research and Vulnerability Assessment**

Extensive CVE research was conducted to identify vulnerabilities relevant to railroad ICS environments and the specific hardware deployed in the testbed. The following Common Vulnerabilities and Exposures informed the design of the 10 attack scenarios:

| CVE ID | Description | CVSS | Impact |
| --- | --- | --- | --- |
| CVE-2022-1161 | Rockwell MicroLogix — Auth Bypass | 9.8 | Remote code execution |
| CVE-2022-1159 | Rockwell Studio 5000 — Malicious Project | 7.7 | Logic manipulation |
| CVE-2020-6992 | Rockwell EtherNet/IP — Memory Corruption | 8.6 | DoS condition |
| CVE-2023-3595 | Rockwell ControlLogix — RCE via CIP | 9.8 | Full system compromise |
| CVE-2019-6553 | Modbus Protocol — No Authentication | N/A | Unauthorized commands |
| CVE-2025-1727 | Railroad Signaling — EoT Weak Auth | 8.1 | Derailment / operational disruption |

*Table 4: CVE Research — Relevant Vulnerabilities for Railroad ICS*

**Attack Criteria Framework**

Each attack scenario was evaluated using an IEEE-compliant methodology incorporating four classification criteria: Attack Vector (Network, Adjacent, Local, Physical), Attack Complexity (Low, High), Impact Categories (Confidentiality, Integrity, Availability, Safety), and Observable Indicators (network traffic, PLC registers, physical behavior). Prevention vectors include network segmentation (VLANs, firewalls, DMZ), authentication (PLC passwords, certificates), monitoring and detection (Zeek IDS, anomaly detection), safety interlocks (watchdog timers, hardwired E-stops), and secure development (code signing, change management).

**Ten Adversarial Scenarios Executed**

The following 10 attack scenarios were designed, documented, and executed against the testbed:

- Attack 1 — Network Reconnaissance: Enumeration of all Modbus-enabled devices on the 10.13.4.0/24 subnet using nmap and Metasploit. Modbus TCP port 502 confirmed open with no authentication required.

- Attack 2 — Modbus Register Read: Unauthorized reading of holding registers and coil values to extract the full operational state of the railroad control system including switch positions, sensor states, and command values.

- Attack 3 — Modbus Coil Write: Direct unauthorized writing to control coils on the OpenPLC master controller, bypassing all ladder logic safety checks. Track switches moved to unauthorized positions via network commands.

- Attack 4 — Modbus Flooding (DoS): Connection flooding of the PowerTrack ESP32 via repeated Modbus TCP requests, causing complete device unresponsiveness. The device remained in a continuous disconnect/reconnect loop with 30+ disconnection events, 20+ timed-out commands, and corrupted connection states.

- Attack 5 — PLC Logic Manipulation: Attempted modification of PLC ladder logic to alter safety interlocks and signaling behavior. This scenario achieved partial success with ongoing research for full implementation.

- Attack 6 — Slave State Spoofing: Man-in-the-middle injection of false sensor readings via Modbus to deceive the PLC into unsafe operating decisions. False occupancy data created collision risk through false situational awareness.

- Attack 7 — Signal Aspect Manipulation: Unauthorized modification of signal states to display incorrect aspects (e.g., green when track is occupied), replicating the Ghost Train collision scenario tied to CVE-2025-1727.

- Attack 8 — Emergency Stop Override: Identification of E-stop input addresses and forced input state via Modbus to disable emergency stop functionality, compromising the primary safety system.

- Attack 9 — Coordinated Multi-Vector: Simultaneous execution of multiple attack types (DoS on PowerTrack, coil write on SwitchTrack, spoofing on Radar) to overwhelm defenses and demonstrate cascading failure.

- Attack 10 — IronSpider Web Attack: Exploitation of PLC web server vulnerabilities to gain initial foothold using malicious web requests. The Railroad Control System dashboard was compromised, providing both a Control Panel and Advanced Controls interface for direct manipulation of all track switches, radar systems, and power controls.

**Attack Platform Summary**

The attacking device was the Raspberry Pi running Kali Linux. While Metasploit was available, no specialized exploit libraries were required—basic scripts using the standard modbusclient module were sufficient to exploit the entire system via Modbus TCP on port 502. The following Modbus function codes were used during attacks: READ_COILS for reconnaissance, READ_HOLDING_REGISTERS for device identification, READ_ID for enumeration, WRITE_COIL as the primary attack vector, and WRITE_COILS (attempted for multi-coil writes).

**Key Research Findings**

Protocol Vulnerability: Modbus TCP has zero authentication—any network device can issue control commands. No specialized exploit code is required; Metasploit’s standard modbusclient module was sufficient for all attack scenarios.

Devices Compromised: All slave devices were affected by Metasploit modbusclient. The OpenPLC master accepted unauthorized coil writes, bypassing all ladder logic safety checks. The PowerTrack ESP32 was taken fully offline via connection flooding, confirming denial-of-service vulnerability.

Physical Impact Confirmed: Track switches moved to unauthorized positions via network commands, confirming cyber-physical impact. The attack maps directly to the Ghost Train Signaling Spoof scenario (CVE-2025-1727), confirming real-world relevance.

**CVSS Score Estimation**

| Metric | Value | Rationale |
| --- | --- | --- |
| Attack Vector | Network | Exploitable remotely over TCP/IP |
| Attack Complexity | Low | No special conditions required |
| Privileges Required | None | No authentication needed |
| User Interaction | None | Fully automated attack |
| Confidentiality Impact | High | All register values readable |
| Integrity Impact | High | All coil values writable |
| Availability Impact | High | Device DoS demonstrated |
| CVSS Score (estimated) | 9.8 CRITICAL | Maximum severity vulnerability |

*Table 6: CVSS Score Estimation*

## 3.2 Problems Encountered/Analysis of Problems Solved

- N-Scale materials and Allen-Bradley MicroLogix PLCs experienced procurement delays, requiring the team to develop primarily on the LEGO/OpenPLC platform initially before transitioning to the Bachmann HO-scale layout.

- OpenPLC Runtime compilation takes approximately one hour on Raspberry Pi. This was mitigated through pre-built images and streamlined deployment procedures.

- GPIO inputs %IX0.0 and %IX0.1 read TRUE by default due to hardware pull-up resistors, requiring logic inversion in the ladder logic programs.

- DIN rail 3D prints required multiple iterations for proper fitment with the Bambu Lab printers before achieving reliable mounting.

- OpenPLC Editor documentation lacked depth; the team synthesized knowledge from multiple sources and conducted extensive independent research.

- ESP32 Modbus TCP slave library had limited documentation for multi-coil configurations, requiring custom implementation.

- PowerTrack ESP32 vulnerability to connection flooding was discovered during testing—the device’s limited TCP stack cannot handle concurrent Modbus connections, creating an inherent DoS vulnerability.

**Mitigation Recommendations**

| Priority | Mitigation | Implementation | Protects Against |
| --- | --- | --- | --- |
| CRITICAL | Network Segmentation | Place all ESP32 slaves on isolated VLAN; only OpenPLC master IP allowed on port 502 | All network-based attacks |
| CRITICAL | Modbus Firewall ACL | Block port 502 from all IPs except 10.13.4.109 (master) | Unauthorized coil writes |
| HIGH | HMAC Authentication | Add Hash-based Message Authentication Code layer to Modbus frames | MITM, replay attacks |
| HIGH | Modbus Write Validation | Master validates all coil changes before forwarding to slaves | Unauthorized commands |
| MEDIUM | IDS Monitoring | Alert when WRITE_COIL originates from non-master IP | Attack detection |
| MEDIUM | Fail-Safe Defaults | Slaves default to safe state on communication timeout | Denial of service |
| LOW | Audit Logging | Log all Modbus transactions with source IP and timestamp | Forensic investigation |
| LOW | Physical Port Security | Lock unused network ports on managed switch | Physical access attacks |

*Table 7: Mitigation Recommendations*

## 3.3 Timeline

| Phase | Period | Activities | Status |
| --- | --- | --- | --- |
| Phase 1: Planning & Design | Jan 12 – Feb 02 | Project proposal, literature review, requirements gathering, system design, component procurement | Complete |
| Phase 2: Build & Integration | Feb 02 – Mar 02 | Track assembly, power distribution, PLC installation, basic wiring, ladder logic programming, I/O testing | Complete |
| Phase 3: Network & Software | Mar 02 – Mar 16 | Network configuration, VLAN setup, dashboard development, Modbus integration | Complete |
| Phase 4: Design Review | Mar 16 – Mar 23 | PDR preparation, documentation, Preliminary Design Review (PDR) | Complete |
| Phase 5: Cyber Testing | Mar 30 – Apr 13 | Attack scenario development, script writing, attack execution, data collection | Complete |
| Phase 6: Analysis & Closeout | Apr 13 – May 08 | Analysis, results documentation, final paper writing, Showtime Demo (May 11) | Complete |

*Table 9: Project Timeline*

## 3.4 Recommendations

- Implement VLAN segmentation to isolate OT slave devices from the broader network, permitting only the OpenPLC master IP on Modbus port 502.

- Deploy HMAC-based authentication for Modbus frames to prevent unauthorized command injection.

- Integrate Zeek IDS with custom Modbus rule sets to alert on WRITE_COIL commands originating from non-master IP addresses.

- Implement fail-safe defaults on all ESP32 slave devices to return to a safe state upon communication timeout.

- Add hardware-level emergency stop circuits that are not software-controllable to prevent E-stop override attacks.

- Develop a standardized ICS training curriculum based on the 10 adversarial scenarios for educational deployment.

# 4. Conclusion

## 4.1 Summary of Project

ICS Rail Testbed successfully demonstrates cyber-physical threat modeling of IT/OT lateral movement in a railroad systems testbed environment, providing a safe, ethical, and cost-effective training platform for cybersecurity professionals. The project achieved 9.5 out of 10 adversarial scenarios designed, documented, and validated. Detection latency of under 50 milliseconds was achieved for OpenPLC response. Physical impact was verified and validated through visible track switch manipulation, radar system disablement, and power control device denial of service. All documentation follows IEEE-compliant procedures and analysis standards. The platform was delivered at a cost of approximately $1,355 compared to $500K+ alternatives, representing over 99% cost savings for a research-grade cyber-physical training capability.

## 4.2 Lessons Learned

Time Management: Balancing coursework with project milestones required careful planning and prioritization of tasks. The dual-track development approach—working on available LEGO/RPi hardware while awaiting industrial components—enabled continuous progress without idle periods.

Teamwork: Cross-functional collaboration between IT, OT, and physical systems expertise was essential for project success. Independent research with knowledge-sharing sessions proved effective for a team with varied experience levels.

Communication: Regular check-ins with advisors and stakeholders kept the project aligned with objectives and on schedule.

Adaptability: Hardware procurement challenges required pivoting approaches while maintaining project goals and timelines. The team successfully transitioned from the initial LEGO prototype to the Bachmann HO-scale layout while preserving the core OpenPLC/Modbus architecture.

Documentation: Thorough documentation enables knowledge transfer and repeatability for future cohorts. The TI/TO-style SOP binder approach ensures that incoming teams can reproduce all experiments and extend the platform.

## 4.3 Future Work

- Integration with machine learning-based anomaly detection for automated identification of unauthorized Modbus commands.

- Expansion to wireless communication attack vectors including Wi-Fi and BLE exploitation of ESP32 slave devices.

- Development of a standardized ICS training curriculum based on the 10 adversarial scenarios for educational deployment across university programs.

- Integration of Allen-Bradley MicroLogix 1100 PLCs with RSLogix 500 programming for full industrial-grade demonstration capability.

- Publication of research findings at UIW Research Week 2027, IEEE ICS Security Conference 2027, and ASEE Conference 2027.

# References

**{IEEE Publication Format}**

[1] R. M. Lee, M. J. Assante, and T. Conway, “Analysis of the cyber attack on the Ukrainian power grid,” E-ISAC, Washington, DC, USA, Rep., Mar. 2016.

[2] E. D. Knapp and J. T. Langill, Industrial Network Security, 2nd ed. Waltham, MA, USA: Syngress, 2015.

[3] IEEE Standard for Local and Metropolitan Area Networks—Bridges and Bridged Networks, IEEE Standard 802.1Q-2022, 2022.

[4] D. Critchlow Jr. and J. A. Montez Jr., “ICS Rail Testbed: Cyber-physical threat modeling of IT/OT lateral movement in railroad control systems,” UIW Engineering Senior Design Proposal, Jan. 2026.

[5] T. J. Williams, “The Purdue Enterprise Reference Architecture,” Comput. Ind., vol. 24, no. 2–3, pp. 141–158, Sep. 1994.

[6] D. Formby, M. Rad, and R. Beyah, “Lowering the barriers to industrial control system security with GRFICS,” in Proc. 2018 USENIX Workshop ASE, Baltimore, MD, USA, Aug. 2018.

[7] OpenPLC Project. OpenPLC Runtime for Raspberry Pi. Accessed: Mar. 10, 2026. [Online]. Available: https://openplcproject.github.io/runtime/raspberry-pi/

[8] The Zeek Project. Zeek Network Security Monitor. Accessed: Mar. 10, 2026. [Online]. Available: https://zeek.org/

[9] K. Stouffer et al., “Guide to operational technology (OT) security,” NIST, SP 800-82 Rev. 3, Sep. 2023.

[10] Programmable Controllers—Part 3: Programming Languages, IEC Standard 61131-3, 2013.

[11] Modbus Application Protocol Specification V1.1b3, Modbus Org., 2012. [Online]. Available: https://modbus.org/specs.php

[12] Cybersecurity and Infrastructure Security Agency, “ICS-CERT Advisories,” CISA, 2024. [Online]. Available: https://www.cisa.gov/news-events/ics-advisories

[13] Rockwell Automation, “Rockwell Automation Trust Center,” 2024. [Online]. Available: https://www.rockwellautomation.com/en-us/trust-center.html

[14] MITRE Corporation, “Common Vulnerabilities and Exposures,” CVE, 2025. [Online]. Available: https://cve.mitre.org/

[15] IEC 62443: Industrial communication networks – Network and system security, International Electrotechnical Commission, 2018.

# Appendix A: Attack Scripts and Code

# [REDACTED] The attack scripts, Metasploit module commands, and the Modbus flooding (DoS) code originally contained in this appendix have been removed from the public release of this report. This repository documents the attack methodology at a conceptual level only. The full appendix is retained in the private/academic version of the report.

# Appendix B: PLC I/O Variable Map and Datasheets

**B.1 Allen-Bradley MicroLogix 1100 Series B Specifications**

- Processor: 1763-L16BWA (16-point I/O)

- Digital Inputs: 10 points at 24V DC

- Digital Outputs: 6 relay outputs

- Communication: EtherNet/IP, Modbus/TCP

- Programming: RSLogix 500 / Micro

- Memory: 8K words user program

**B.2 ESP-32 Microcontroller Specifications**

- Wi-Fi and BLE operate in a 2.4 GHz frequency range

- Wi-Fi range of 100–300 ft (30–90 m) in typical indoor environments

- BT Low Energy range of up to 164 ft (50 m) in open spaces

- GPIO pins are not 5V tolerant; operates at 3.3V logic levels

- Supports SPI, I2C, UART, DAC, ADC, and touch sensor interfaces

**B.3 Raspberry Pi 4B Specifications**

- Broadcom BCM2711, Quad-core Cortex-A72 (ARM v8) 64-bit SoC at 1.8GHz

- Available in 1GB, 2GB, 4GB, or 8GB LPDDR4-3200 SDRAM configurations

- 2.4 GHz and 5.0 GHz IEEE 802.11ac wireless, Bluetooth 5.0, BLE

- Gigabit Ethernet

- Standard 40-pin GPIO header (fully backwards compatible with previous boards)

- Operating temperature: 0–50 degrees C ambient

**B.4 DIN Rail Power Supply Specifications**

- Selec RPS120-24-CU: Input 100–240VAC 50/60Hz, Output 24VDC 5A 120W

- EDR-120-12: Input 100–240VAC, Output 12VDC 10A 120W

- Both units are DIN rail mountable with industrial-grade enclosures
