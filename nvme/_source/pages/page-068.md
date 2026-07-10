---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 68
headings: []
---

# Source page 68

NVM Express® Base Specification, Revision 2.1

46
Figure 31: Log Page Support Requirements
Log Page Name Log Page
Identifier
Controller Support Requirements 1 Reference
I/O Administrative Discovery
Telemetry Host-Initiated 07h O O P 5.1.12.1.8
Telemetry Controller-Initiated 08h O O8 P 5.1.12.1.9
Endurance Group Information 09h O  P 5.1.12.1.10
Predictable Latency Per NVM Set 0Ah O O8 P 5.1.12.1.11
Predictable Latency Event Aggregate 0Bh O O8 P 5.1.12.1.12
Asymmetric Namespace Access 0Ch O P P 5.1.12.1.13
Persistent Event 0Dh O O P 5.1.12.1.14
LBA Status Information 0Eh Refer to the NVM Command Set Specification
Endurance Group Event Aggregate 0Fh O O P 5.1.12.1.15
Media Unit Status 10h O2 P P 5.1.12.1.16
Supported Capacity Configuration List 11h O2 P P 5.1.12.1.17
Feature Identifiers Supported and
Effects 12h M3 M3,9 M9 5.1.12.1.18
NVMe-MI Commands Supported and
Effects 13h M3,7 M3,7 O 5.1.12.1.19
Command and Feature Lockdown 14h O O P 5.1.12.1.20
Boot Partition 15h O O P 5.1.12.1.21
Rotational Media Information 16h O P P 5.1.12.1.22
Dispersed Namespace Participating
NVM Subsystems 17h O O P 5.1.12.1.23
Management Address List 18h O O O 5.1.12.1.24
Physical Interface Receiver Eye
Opening Measurement 19h O4 O4 P Note 11
Reachability Groups 1Ah M6 P P 5.1.12.1.25
Reachability Associations 1Bh M6 P P 5.1.12.1.26
Changed Allocated Namespace List 1Ch O O P 5.1.12.1.27
FDP Configurations 20h O5 P P 5.1.12.1.28
Reclaim Unit Handle Usage 21h O5 P P 5.1.12.1.29
FDP Statistics 22h O5 P P 5.1.12.1.30
FDP Events 23h O5 P P 5.1.12.1.31
Discovery 70h P P M 5.1.12.3.1
Host Discovery 71h P P O 5.1.12.3.2
AVE Discovery 72h P P O 5.1.12.3.3
Pull Model DDC Request 73h P P M10 5.1.12.3.4
