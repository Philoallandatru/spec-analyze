---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 81
headings: []
---

# Source page 81

NVM Express® Base Specification, Revision 2.1

59
Figure 41: Offset 14h: CC – Controller Configuration
Bits Type Reset Description
06:04 RW 000b
I/O Command Set Selected (CSS): This field specifies the I/O Command Set
or Sets that are selected. This field shall only be changed when the controller is
disabled (i.e., CC.EN is cleared to ‘0’). The I/O Command Set or Sets that are
selected shall be used for all I/O Submission Queues.
Value Definition
000b
CAP.CSS.NCSS
Bit Definition
1b NVM Command Set
0b Reserved

001b to 101b Reserved
110b
CAP.CSS.IOCSS
Bit Definition
1b
All Supported I/O Command Sets
The I/O Command Sets that are
supported are reported in the
Identify I/O Command Set data
structure (refer to section
5.1.13.2.19).
0b Reserved

111b
CAP.CSS.NOIOCSS
Bit Definition
1b
Admin Command Set only
I/O Command Set and I/O
Command Set specific Admin
commands are not supported.
Any I/O Command Set
specific Admin command
submitted on the Admin
Submission Queue is aborted
with a status code of Invalid
Command Opcode.
0b Reserved

For Discovery controllers, this property shall be cleared to 000b.
03:01 RO 000b Reserved
