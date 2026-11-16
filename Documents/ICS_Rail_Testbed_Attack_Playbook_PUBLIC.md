**ATTACK SCENARIO PLAYBOOK**

10 Adversarial Scenarios for Railroad ICS/OT

ICS Rail Testbed — CVE Research & Exploitation Analysis

**AUTHORIZED LAB USE ONLY — Version 1.0 | 25 March 2026**

# 1. CVE Research Summary

The following CVEs are directly relevant to railroad critical infrastructure and ICS/SCADA systems. Each has been researched against CISA ICS advisories, NVD records, and vendor security bulletins. These vulnerabilities form the technical basis for the 10 attack scenarios that follow.

| CVE ID | Affected System | CVSS | Description & Rail Impact |
| --- | --- | --- | --- |
| CVE-2025-1727 | EOT/HOT Protocol | 8.1 | Weak authentication in End-of-Train/Head-of-Train device radio protocol allows unauthorized emergency brake commands via adjacent RF access. Affects railroad rolling stock across multiple manufacturers (Wabtec, Siemens, DPS). |
| CVE-2017-14472 | AB MicroLogix 1400 | 10.0 | Unauthenticated remote modification of ladder logic via PCCC protocol. No authentication required for program upload/download, enabling complete control logic replacement. |
| CVE-2017-12088 | AB MicroLogix 1400 | 8.6 | Specially crafted Ethernet packet causes PLC fault state and potential ladder logic deletion. Remote, unauthenticated attacker can render controller inoperable. |
| CVE-2020-6984 | AB MicroLogix 1100/1400 | 7.5 | Credential disclosure via unauthenticated Ethernet/IP requests returns management passwords in plaintext, enabling subsequent authenticated attacks. |
| CVE-2017-12089 | AB MicroLogix 1400 | 6.8 | SNMP-set request without firmware flash commands causes device power cycle reboot. Enables repeated availability disruption. |
| CVE-2017-16740 | AB MicroLogix 1400 | 9.8 | Stack-based buffer overflow in web server request handling allows remote code execution. Attacker can execute arbitrary code on PLC processor. |
| CVE-2021-22659 | AB MicroLogix 1400 | 7.5 | Crafted Modbus TCP packet enables register read/write manipulation causing buffer overflow and denial of service. Affects Modbus-enabled controllers. |
| CVE-2022-1159 | Rockwell Studio 5000 | 7.7 | Malicious DLL injection in engineering workstation enables ladder logic modification during compilation without operator awareness. |
| CVE-2023-3595 | Rockwell ControlLogix | 9.8 | Remote code execution via CIP protocol exploitation. Enables complete PLC compromise from network access with no authentication. |
| CVE-2024-6242 | Rockwell ControlLogix | 8.4 | Authentication bypass allows CIP commands that modify PLC configuration and user projects. Trusted Slot feature can be circumvented. |

# 2. Attack Criteria Matrix

Each attack is evaluated against five criteria: Feasibility (can we execute it in the lab), Observability (can Zeek/logs detect it), Impact (physical consequence on rail operations), Complexity (skill level required), and Preventability (what stops the attack vector).

| ID | Attack Name | Feasible | Observable | Impact | Complexity | Primary Prevention |
| --- | --- | --- | --- | --- | --- | --- |
| ATK-01 | Ghost Train Spoof | HIGH | HIGH | CRITICAL | Medium | Keyswitch + CIP Security |
| ATK-02 | Modbus Register Manip | HIGH | HIGH | HIGH | Low | Function code allowlist |
| ATK-03 | PLC Logic Wipe | HIGH | HIGH | CRITICAL | Low | Firmware patch + IDS |
| ATK-04 | Web Session Hijack | MEDIUM | MEDIUM | HIGH | High | Disable web server |
| ATK-05 | Default Credential | HIGH | HIGH | CRITICAL | Low | Password change policy |
| ATK-06 | RF Brake Injection | LOW* | LOW | CRITICAL | High | Protocol encryption |
| ATK-07 | Ransomware Sim | HIGH | HIGH | HIGH | Medium | Segmentation + backups |
| ATK-08 | Power Cycle Loop | HIGH | HIGH | HIGH | Low | Disable SNMP / v3 auth |
| ATK-09 | Backup Trojan | MEDIUM | LOW | CRITICAL | High | Logic integrity checks |
| ATK-10 | SSH Lateral Pivot | HIGH | HIGH | HIGH | Medium | No dual-homed hosts |

* ATK-06 RF simulation is approximated in the lab using Modbus commands to simulate brake actuation signal.

# 3. Detailed Attack Scenarios

## ATK-01: Ghost Train Signaling Spoof

| Parameter | Details |
| --- | --- |
| CVE Basis | CVE-2017-14472 |
| Category | Integrity |
| MITRE ATT&CK ICS | T0831 (Manipulation of Control) |
| Description | Modify PLC ladder logic to report clear track while train is present, creating collision risk. |
| Entry Point | Compromised engineering workstation on IT network |
| Pivot Method | PCCC protocol to PLC Ethernet port (no auth) |
| Adversarial Action | Write False to signal output (%QX0.0) and occupancy bit |
| Physical Impact | Signal shows Green while track is occupied. Potential rear-end collision. |
| Detection (Zeek/Logs) | Zeek: modbus.log FC5 write commands from unauthorized source IP. Auditd: PLC logic change events. |
| Prevention Controls | Physical keyswitch in RUN mode, CIP Security TLS, network micro-segmentation, Purdue Level 1 firewall. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-02: Modbus Register Manipulation

| Parameter | Details |
| --- | --- |
| CVE Basis | CVE-2021-22659 |
| Category | Integrity |
| MITRE ATT&CK ICS | T0855 (Unauthorized Command Message) |
| Description | Send crafted Modbus TCP packets to read/write arbitrary register values, corrupting sensor data and control outputs. |
| Entry Point | Network access to VLAN 20 (OT segment) |
| Pivot Method | Direct Modbus TCP connection to port 502 |
| Adversarial Action | Write arbitrary values to holding registers and coils via FC5/FC6/FC16 |
| Physical Impact | Sensor readings falsified, actuator states corrupted, safety interlocks bypassed. |
| Detection (Zeek/Logs) | Zeek: anomalous Modbus function codes, write commands to read-only registers, high packet rate. |
| Prevention Controls | Modbus-aware firewall (function code allowlisting), VLAN isolation, read-only register configuration. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-03: PLC Logic Wipe (Permanent DoS)

| Parameter | Details |
| --- | --- |
| CVE Basis | CVE-2017-12088 |
| Category | Availability |
| MITRE ATT&CK ICS | T0809 (Data Destruction) |
| Description | Send malformed download packet to force PLC into fault state, deleting ladder logic and halting all automation. |
| Entry Point | Network access to PLC Ethernet port |
| Pivot Method | Crafted incomplete download sequence packet |
| Adversarial Action | Send packet mimicking download initiation without completion signal |
| Physical Impact | PLC enters FAULT state, ladder logic deleted. Manual restore required from backup. Total system halt. |
| Detection (Zeek/Logs) | Zeek: malformed PCCC packets, PLC state change alerts. Rsyslog: fault state transition logged. |
| Prevention Controls | Physical keyswitch, firmware update to patched version, IDS signature for malformed download packets. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-04: Web Portal Session Hijack

| Parameter | Details |
| --- | --- |
| CVE Basis | CVE-2017-16740 |
| Category | Integrity |
| MITRE ATT&CK ICS | T0866 (Exploitation of Remote Services) |
| Description | Exploit PLC embedded web server buffer overflow to execute arbitrary code or hijack engineer session. |
| Entry Point | Malicious JavaScript injected into PLC web admin portal |
| Pivot Method | Engineer browser session on IT workstation |
| Adversarial Action | XSS payload captures session token, then issues authenticated commands to PLC |
| Physical Impact | Attacker controls PLC through hijacked legitimate session. Track switches toggled during transit. |
| Detection (Zeek/Logs) | Zeek: HTTP anomaly detection on port 80/443 to PLC. Log server: unusual web API calls. |
| Prevention Controls | Disable embedded web server, session timeout enforcement, Content-Security-Policy headers. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-05: Default Credential Takeover

| Parameter | Details |
| --- | --- |
| CVE Basis | CVE-2020-6984 |
| Category | Initial Access |
| MITRE ATT&CK ICS | T0859 (Valid Accounts) |
| Description | Extract management credentials via unauthenticated Ethernet/IP request, gaining full admin access. |
| Entry Point | Network scan discovers PLC on OT VLAN |
| Pivot Method | CIP Identity request reveals cleartext password |
| Adversarial Action | Use extracted credentials to log into OpenPLC dashboard or HMI |
| Physical Impact | Full administrative control over PLC: modify logic, change setpoints, disable safety interlocks. |
| Detection (Zeek/Logs) | Zeek: CIP identity requests from non-engineering IPs. Pi-hole: DNS queries to suspicious domains. |
| Prevention Controls | Change all default credentials, disable cleartext protocol responses, implement certificate-based auth. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-06: Braking System RF Injection

| Parameter | Details |
| --- | --- |
| CVE Basis | CVE-2025-1727 |
| Category | Safety |
| MITRE ATT&CK ICS | T0830 (Man in the Middle) |
| Description | Simulate unauthorized emergency brake command via End-of-Train device protocol weakness. |
| Entry Point | Adjacent RF access to train communication frequency |
| Pivot Method | Unauthenticated EOT/HOT radio protocol |
| Adversarial Action | Transmit forged emergency stop packet to braking system receiver |
| Physical Impact | Unexpected emergency braking at speed causes cargo shift, mechanical failure, derailment risk. |
| Detection (Zeek/Logs) | RF monitoring equipment, Zeek: anomalous serial-to-IP gateway traffic patterns. |
| Prevention Controls | Protocol upgrade (AAR RESC in progress), RF encryption, authenticated command validation. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-07: Ransomware Propagation Simulation

| Parameter | Details |
| --- | --- |
| CVE Basis | N/A (WannaCry class) |
| Category | Availability |
| MITRE ATT&CK ICS | T0828 (Loss of Productivity and Revenue) |
| Description | Simulate ransomware spreading from IT network to OT-adjacent systems, blinding dispatch operators. |
| Entry Point | Phishing email on IT workstation |
| Pivot Method | SMB lateral movement to engineering workstation |
| Adversarial Action | Encrypt HMI databases and dispatch screens, display ransom note |
| Physical Impact | Operators lose all visibility into train positions and track status. Manual operations required. |
| Detection (Zeek/Logs) | Pi-hole: C2 domain blocking. Zeek: SMB lateral movement patterns. Log server: mass file encryption events. |
| Prevention Controls | Network segmentation, unidirectional gateway/data diode, offline backups, endpoint detection. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-08: PLC Power Cycle Loop

| Parameter | Details |
| --- | --- |
| CVE Basis | CVE-2017-12089 |
| Category | Availability |
| MITRE ATT&CK ICS | T0816 (Device Restart/Shutdown) |
| Description | Continuous SNMP-set requests force PLC into endless reboot cycle, preventing stable operation. |
| Entry Point | Network access to PLC management interface |
| Pivot Method | SNMP protocol (default community string) |
| Adversarial Action | Rapid transmission of SNMP-set requests triggering firmware flash sequence without actual flash |
| Physical Impact | PLC oscillates between boot and fault states. No stable automation possible. Rail system seized. |
| Detection (Zeek/Logs) | Zeek: high-rate SNMP traffic to PLC. Log server: repeated boot cycle log entries. |
| Prevention Controls | Disable SNMP or change community strings, SNMP v3 with authentication, rate limiting on switch. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-09: Backup Module Logic Trojan

| Parameter | Details |
| --- | --- |
| CVE Basis | CVE-2017-14472 (variant) |
| Category | Persistence |
| MITRE ATT&CK ICS | T0839 (Module Firmware) |
| Description | Write malicious ladder logic to PLC backup memory module, surviving power cycles and manual resets. |
| Entry Point | Prior PLC access via any exploitation method |
| Pivot Method | Unauthenticated write to backup memory |
| Adversarial Action | Upload modified logic to backup module that intermittently toggles track switches |
| Physical Impact | Persistent backdoor survives resets. Intermittent switch failures appear as hardware malfunctions. |
| Detection (Zeek/Logs) | Logic comparison: hash current PLC program against known-good baseline. Audit backup module contents. |
| Prevention Controls | Physical write-protect on memory modules, regular logic integrity verification, trusted firmware. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

## ATK-10: Lateral Pivot via SSH Tunnel

| Parameter | Details |
| --- | --- |
| CVE Basis | N/A (technique) |
| Category | Lateral Movement |
| MITRE ATT&CK ICS | T0812 (Default Credentials) + T0886 (Remote Services) |
| Description | Demonstrate multi-hop attack from IT network through engineering workstation to OT PLC. |
| Entry Point | Compromised IT workstation (VLAN 10) |
| Pivot Method | SSH tunnel through dual-homed engineering host |
| Adversarial Action | Establish SOCKS proxy, route Modbus commands through tunnel to bypass VLAN segmentation |
| Physical Impact | Attacker on IT network directly controls OT devices. VLAN segmentation bypassed by application-layer tunnel. |
| Detection (Zeek/Logs) | Zeek: SSH tunnel detection (long-duration sessions, unusual data volumes). Log server: SSH auth from unexpected IPs. |
| Prevention Controls | Prohibit dual-homed hosts, jump box with MFA, network TAP monitoring, session recording. |
| Lab Execution Script | [REDACTED — lab execution command removed from public release] |

# 4. Defense-in-Depth Framework

The following tiered defense model maps preventive controls to each attack vector. Each tier builds on the previous, providing cumulative protection.

| Tier | Name | Technical Controls | Attacks Mitigated |
| --- | --- | --- | --- |
| I | Basic Hygiene | Change default passwords, disable unused services/protocols, apply firmware patches, physical keyswitch management. | ATK-05, ATK-08 |
| II | Network Segmentation | VLAN isolation (802.1Q), firewall between IT/OT zones, Modbus function code allowlisting, MAC port security. | ATK-01, ATK-02, ATK-10 |
| III | Monitoring & Detection | Zeek IDS with Modbus parser, centralized syslog, Pi-hole DNS sinkhole, auditd system call monitoring, PCAP archival. | ATK-01 through ATK-10 (detection) |
| IV | Advanced Hardening | CIP Security (TLS/DTLS), unidirectional data diode, Safety Instrumented System (SIS), logic integrity verification, MFA for engineering access. | ATK-03, ATK-04, ATK-06, ATK-09 |
