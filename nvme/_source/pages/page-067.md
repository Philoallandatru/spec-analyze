---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 67
headings: ["3.1.3.5 Log Page Support Requirements"]
---

# Source page 67

NVM Express® Base Specification, Revision 2.1

45
Figure 29: Fabrics Command Support Requirements
Command Opcode
Value
Fabrics
Command
Type
Controller Support Requirements1, 2 Reference
I/O Administrative Discovery
Vendor Specific
Vendor Specific 7Fh C0h to FFh O  O O
Notes:
1. O/M/P definition: O = Optional, M = Mandatory, P = Prohibited
2. For NVMe over PCIe implementations, all Fabrics commands are prohibited. For NVMe over Fabrics
implementations, the commands are as noted in the table.

Figure 30: Common I/O Command Support Requirements
Command
Combined
Opcode
Value
Controller Support Requirements 1 Reference
I/O Administrative Discovery
Flush 00h M P P 7.2
Reservation Register 0Dh O 2 P P 7.6
Reservation Report 0Eh O 2 P P 7.8
Reservation Acquire 11h O 2 P P 7.5
I/O Management Receive 12h O 3 P P 7.3
Reservation Release 15h O 2 P P 7.7
Cancel 18h O P P 7.1
I/O Management Send 1Dh O 3 P P 7.4
Vendor Specific
Vendor Specific  O O O
Notes:
1. O/M/P definition: O = Optional, M = Mandatory, P = Prohibited
2. Mandatory if reservations are supported as indicated in the Identify Controller data structure.
3. Mandatory for controllers that support the Flexible Data Placement capability (refer to section 8.1.10). Refer to
the FDPS bit in Figure 312.
 Log Page Support Requirements
Figure 31 defines log pages that are mandatory, optional, and prohibited for an I/O controller, Administrative
controller, and Discovery controller. I/O Command Set specific log page support requirements are
described within individual I/O Command Set specifications.
Figure 31: Log Page Support Requirements
Log Page Name Log Page
Identifier
Controller Support Requirements 1 Reference
I/O Administrative Discovery
Supported Log Pages 00h M3 M3 M 5.1.12.1.1
Error Information 01h M M O 5.1.12.1.2
SMART / Health Information
(Controller scope) 02h M O P
5.1.12.1.3 SMART / Health Information
(Namespace scope) 02h O O P
Firmware Slot Information 03h M O P 5.1.12.1.4
Changed Attached Namespace List 04h O O P 5.1.12.1.5
Commands Supported and Effects 05h M3 M M 5.1.12.1.6
Device Self-test 06h O O P 5.1.12.1.7
