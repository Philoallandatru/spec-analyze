---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 70
headings: []
---

# Source page 70

NVM Express® Base Specification, Revision 2.1

48
Figure 32: Feature Support Requirements
Feature Name Feature
Identifier
Controller Support Requirements 1 Reference
I/O Administrative Discovery
Host Controlled Thermal Management 10h O O P 5.1.25.1.9
Non-Operational Power State Config 11h O O P 5.1.25.1.10
Read Recovery Level Config 12h O O P 5.1.25.1.11
Predictable Latency Mode Config 13h O O9 P 5.1.25.1.12
Predictable Latency Mode Window 14h O O9 P 5.1.25.1.13
LBA Status Information Report Interval 15h Refer to the NVM Command Set Specification
Host Behavior Support 16h O O P 5.1.25.1.14
Sanitize Config 17h O O9 P 5.1.25.1.15
Endurance Group Event Configuration 18h O O9 P 5.1.25.1.16
I/O Command Set Profile 19h O P P 5.1.25.1.17
Spinup Control 1Ah O P P 5.1.25.1.18
Power Loss Signaling Config 1Bh O O P 5.1.25.1.19
Performance Characteristics 1Ch Refer to the NVM Command Set Specification
Flexible Data Placement 1Dh O6 P P 5.1.25.1.20
Flexible Data Placement Events 1Eh O6 P P 5.1.25.1.21
Namespace Admin Label 1Fh O O P 5.1.25.1.22
Key Value Configuration 20h Refer to the Key Value Command Set Specification
Controller Data Queue 21h O O P 5.1.25.1.23
Embedded Management Controller
Address 78h O O O 5.1.25.1.24
Host Management Agent Address 79h O O O 5.1.25.1.25
Enhanced Controller Metadata 7Dh O5 O5 O 5.1.25.1.26.1
Controller Metadata 7Eh O5 O5 O 5.1.25.1.26.2
Namespace Metadata 7Fh O5 O5 O 5.1.25.1.26.3
Software Progress Marker 80h O O P 5.1.25.1.27
Host Identifier 81h O3 O P 5.1.25.1.28
Reservation Notification Mask 82h O4 P P 5.1.25.1.29
Reservation Persistence 83h O4 P P 5.1.25.1.30
