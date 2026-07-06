---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 69
headings: ["3.1.3.6 Feature Support Requirements"]
---

# Source page 69

NVM Express® Base Specification, Revision 2.1

47
Figure 31: Log Page Support Requirements
Log Page Name Log Page
Identifier
Controller Support Requirements 1 Reference
I/O Administrative Discovery
Reservation Notification 80h O P P 5.1.12.1.32
Sanitize Status 81h O O8 P 5.1.12.1.33
Program List 82h Refer to the Computational Programs Command Set
Specification Downloadable Program Types List 83h
Memory Range Set List 84h
 85h to BEh Refer to the applicable I/O Command Set specification
Changed Zone List BFh Refer to the Zoned Namespace Command Set
Specification
Vendor Specific C0h to FFh
Notes:
1. O/M/P definition: O = Optional, M = Mandatory, P = Prohibited
2. Mandatory for controllers that support Fixed Capacity Management (refer to section 8.1.4.2).
3. Optional for NVM Express revision 1.4 and earlier.
4. If this log page is not described for a specific physical interface (refer to the applicable NVM Express transport
specification), then this log page is prohibited for that transport.
5. Mandatory for controllers that support the Flexible Data Placement capability (refer to section 8.1.10). Refer to
the FDPS bit in Figure 312.
6. Optional for controllers that do not support Reachability Reporting (refer to section 8.1.19).
7. Optional if the NVMe-MI Send command and the NVMe-MI Receive command are not supported (refer to Figure
28).
8. Prohibited for an Exported NVM subsystem (refer to section 8.3.3).
9. Optional if the Set Features command is not supported (refer to Figure 28).
10. Mandatory for CDCs and prohibited for Discovery controllers that are not a CDC.
11. Refer to the applicable NVM Express transport specification.
 Feature Support Requirements
Figure 32 defines features that are mandatory, optional, and prohibited for an I/O controller, Administrative
controller, and Discovery controller . If any feature is supported, then the Set Features command and the
Get Features command shall be supported. I/O Command Set specific feature support requirements are
described within individual I/O Command Set specifications.
Figure 32: Feature Support Requirements
Feature Name Feature
Identifier
Controller Support Requirements 1 Reference
I/O Administrative Discovery
Arbitration 01h M P P 5.1.25.1.1
Power Management 02h M O P 5.1.25.1.2
LBA Range Type 03h Refer to the NVM Command Set Specification
Temperature Threshold 04h M O P 5.1.25.1.3
Error Recovery 05h Refer to the NVM Command Set Specification
Volatile Write Cache 06h O P P 5.1.25.1.4
Number of Queues 07h M P P 5.1.25.2.1
Interrupt Coalescing 08h Note 2 Note 2 P 5.1.25.2.2
Interrupt Vector Configuration 09h Note 2 Note 2 P 5.1.25.2.3
Write Atomicity Normal 0Ah Refer to the NVM Command Set Specification
Asynchronous Event Configuration 0Bh M O8 M10 5.1.25.1.5
Autonomous Power State Transition 0Ch O O P 5.1.25.1.6
Host Memory Buffer 0Dh O O P 5.1.25.2.4
Timestamp 0Eh O O P 5.1.25.1.7
Keep Alive Timer 0Fh M7 M7 M10 5.1.25.1.8
