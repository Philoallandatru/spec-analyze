---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 394
headings: []
---

# Source page 394

NVM Express® Base Specification, Revision 2.1

372
Figure 385: Set Features – Feature Identifiers
Feature
Identifier
Current Setting
Persists
Across Power
Cycle and
Reset 2
Uses
Memory
Buffer for
Attributes
Feature Name Scope 6
0Dh No 3 No 4 Host Memory Buffer Controller
0Eh No Yes Timestamp Controller
0Fh No No Keep Alive Timer Controller
10h Yes No Host Controlled Thermal Management Controller
11h No No Non-Operational Power State Config Controller
12h Yes No Read Recovery Level Config NVM Set
NVM subsystem
13h No Yes Predictable Latency Mode Config NVM Set
14h No No Predictable Latency Mode Window NVM Set
15h Refer to the NVM Command Set Specification
16h No Yes Host Behavior Support Controller
17h Yes No Sanitize Config NVM subsystem
18h No No Endurance Group Event Configuration Endurance Group
19h Yes No I/O Command Set Profile Controller
1Ah Yes No Spinup Control NVM subsystem
1Bh Yes No Power Loss Signaling Config Domain
1Ch Refer to the NVM Command Set Specification
1Dh Yes No Flexible Data Placement Endurance Group
1Eh Yes Yes Flexible Data Placement Events Reclaim Unit
Handle
1Fh Yes Yes Namespace Admin Label Namespace
20h Refer to the Key Value Command Set Specification
21h No Yes Controller Data Queue
Controller Data
Queue per
controller
22h to 77h Reserved
78h Yes Yes Embedded Management Controller Address NVM subsystem
79h Yes Yes Host Management Agent Address NVM subsystem
7Ah to 7Ch Reserved for Management Features.
7Dh No Yes Enhanced Controller Metadata Controller
7Eh No Yes Controller Metadata Controller
7Fh No Yes Namespace Metadata Namespace per
controller
80h Yes No Software Progress Marker Controller
81h No Yes Host Identifier Controller
82h No No Reservation Notification Mask Namespace
83h Yes No Reservation Persistence Namespace
84h No No Namespace Write Protection Config Namespace
85h No No Boot Partition Write Protection Config Controller
86h to BFh Reserved
