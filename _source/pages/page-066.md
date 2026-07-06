---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 66
headings: []
---

# Source page 66

NVM Express® Base Specification, Revision 2.1

44
Figure 28: Admin Command Support Requirements
Command
Combined
Opcode
Value
Controller Support Requirements1 Reference
I/O Administrative Discovery
Fabric Zoning Send 29h P P M 7 5.3.6
Create Exported NVM Subsystem 2Ah P O P 5.3.2
Manage Exported NVM Subsystem 2Dh P O P 5.3.8
Manage Exported Namespace 31h P O P 5.3.7
Manage Exported Port 35h P O P 5.3.9
Send Discovery Log Page 39h P P M 7 5.3.10
Track Send 3Dh O O P 5.1.26
Track Receive 3Eh O O P 5.1.27
Migration Send 41h O O P 5.1.16
Migration Receive 42h O O P 5.1.17
Controller Data Queue 45h O O P 5.1.4
Doorbell Buffer Config 7Ch O O P 5.2.5
I/O Command Set specific Admin commands
Format NVM 80h O O P 5.1.10
Security Send 81h O O P 5.1.24
Security Receive 82h O O P 5.1.23
Sanitize 84h O 3 O 3 P 5.1.22
I/O Command Set specific Admin
command
85h
Note 8 P P Note 8 86h
88h
89h
Vendor Specific
Vendor Specific  O O O
Notes:
1. O/M/P definition: O = Optional, M = Mandatory, P = Prohibited
2. Prohibited if the Keep Alive Timer feature is not supported (refer to section 3.9).
3. Prohibited for an Exported NVM Subsystem (refer to section 8.3.3).
4. Mandatory if any of the features in Figure 32 are implemented.
5. Mandatory if Telemetry Log, Firmware Commit, or SMART/Health Critical Warnings are supported.
6. For Discovery controllers that support explicit persistent connections, this command is mandatory. For Discovery
controllers that do not support explicit persistent connections, this command is prohibited.
7. Mandatory for CDCs and optional for Discovery controllers that are not a CDC.
8. Refer to the applicable I/O Command Set specification.
9. Mandatory for NVMe over PCIe. This command is not supported for NVMe over Fabrics.

Figure 29: Fabrics Command Support Requirements
Command Opcode
Value
Fabrics
Command
Type
Controller Support Requirements1, 2 Reference
I/O Administrative Discovery
Property Set
7Fh
00h M M M 6.6
Connect 01h M M M 6.3
Property Get 04h M M M 6.5
Authentication Send 05h O O O 6.2
Authentication Receive 06h O O O 6.1
Disconnect 08h O P P 6.4
