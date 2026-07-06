---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 226
headings: []
---

# Source page 226

NVM Express® Base Specification, Revision 2.1

204
 Figure 202: Get Log Page – Log Page Identifiers
Log Page
Identifier CSI8 Scope Log Page Name Reference
Section
00h Y Controller Supported Log Pages 5.1.12.1.1
01h N Controller Error Information 5.1.12.1.2
02h N Controller / Namespace 1, 2 SMART / Health Information 5.1.12.1.3
03h N Domain / NVM subsystem 6 Firmware Slot Information 5.1.12.1.4
04h N Controller Changed Attached Namespace List 5.1.12.1.5
05h Y Controller Commands Supported and Effects 5.1.12.1.6
06h N
Controller / Domain / NVM
subsystem 3, 4, 6 Device Self-test 5 5.1.12.1.7
07h N Controller / NVM subsystem 7 Telemetry Host-Initiated 5 5.1.12.1.8
08h N Controller / NVM subsystem 7 Telemetry Controller-Initiated 5 5.1.12.1.9
09h N Domain / NVM subsystem 6 Endurance Group Information 5.1.12.1.10
0Ah N Domain / NVM subsystem 6 Predictable Latency Per NVM Set 5.1.12.1.11
0Bh N Domain / NVM subsystem 6 Predictable Latency Event Aggregate 5.1.12.1.12
0Ch N Controller Asymmetric Namespace Access 5.1.12.1.13
0Dh N NVM subsystem Persistent Event Log 5 5.1.12.1.14
0Eh Refer to the NVM Command Set Specification
0Fh N Domain / NVM subsystem 6 Endurance Group Event Aggregate 5.1.12.1.15
10h N Domain / NVM subsystem 5, 6 Media Unit Status 5.1.12.1.16
11h N Domain / NVM subsystem 6 Supported Capacity Configuration List 5.1.12.1.17
12h Y Controller Feature Identifiers Supported and Effects 5.1.12.1.18
13h N Controller NVMe-MI Commands Supported and
Effects 5.1.12.1.19
14h Y NVM subsystem Command and Feature Lockdown 5 5.1.12.1.20
15h N Controller Boot Partition 5.1.12.1.21
16h N Endurance Group Rotational Media Information 5.1.12.1.22
17h N Namespace Dispersed Namespace Participating
NVM Subsystems 5.1.12.1.23
18h N NVM subsystem Management Address List 5.1.12.1.24
19h Refer to the applicable NVM Express transport specification
1Ah N Controller Reachability Groups 5.1.12.1.25
1Bh N Controller Reachability Associations 5.1.12.1.26
1Ch N Controller Changed Allocated Namespace List 5.1.12.1.27
1Dh to 1Fh Reserved
20h N Endurance Group FDP Configurations 5.1.12.1.28
21h N Endurance Group Reclaim Unit Handle Usage 5.1.12.1.29
22h N Endurance Group FDP Statistics 5.1.12.1.30
23h N Endurance Group FDP Events 5.1.12.1.31
24h to 6Fh Reserved
70h N Controller Discovery 5.1.12.3.1
71h N Controller Host Discovery 5.1.12.3.2
72h N Controller AVE Discovery 5.1.12.3.3
73h N Controller Pull Model DDC Request 5.1.12.3.4
74h to 7Fh Reserved
80h N Controller Reservation Notification 5.1.12.1.32
81h N NVM subsystem Sanitize Status 5.1.12.1.33
