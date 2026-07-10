---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 271
headings: []
---

# Source page 271

NVM Express® Base Specification, Revision 2.1

249
Figure 245: Feature Persistent Event Logging Requirements
Feature Name Feature
Identifier
Controller PE Logging Requirements 1
I/O Admin Discovery
Flexible Data Placement 1Dh O P P
Flexible Data Placement Events 1Eh O P P
Namespace Admin Label 1Fh O O P
 20h Refer to the applicable I/O Command Set specification
Embedded Management Controller
Address 78h O O O
Host Management Agent Address 79h O O O
Enhanced Controller Metadata 7Dh O O O
Controller Metadata 7Eh O O O
Namespace Metadata 7Fh O O O
Software Progress Marker 80h NR NR P
Host Identifier 81h O O P
Reservation Notification Mask 82h O P P
Reservation Persistence 83h O P P
Namespace Write Protection Config 84h O O P
Boot Partition Write Protection Config 85h O O P
Notes:
1. O/M/P/NR definition: O = Optional, M = Mandatory, P = Prohibited, NR = Not Recommended
The Command Dwords and Memory Buffer logged in the Set Feature Event data use the same formats as
the formats defined by the Set Features and Get Features commands.
Figure 246: Set Feature Event Data Format
Bytes Description
03:00
Set Feature Event Layout (SFEL): Defines the number of Command Dwords and
the amount of data in the Memory Buffer from the Set Features command
associated with this event.
Bits Description
31:16
Memory Buffer Count (MBC): Defines the number of bytes from the
memory buffer that are logged in the Memory Buffer field. A value of
0h indicates that the Memory Buffer field does not exist.
15:04 Reserved
03
Logged Command Completion Dword 0 (LCCDW0): If this bit is set
to ‘1’, then Dword 0 of the command completion for the Set Features
command is included in the log. If this bit is cleared to ‘0’, then Dword
0 of the command completion command for the Set Features
command is not included in the log.
02:00
Dword Count (DWC): contains the number of consecutive Dwords
starting with Dword 10 from the Set Features command  that are
reported in the Command Dwords field. The values 0h and 7h are
reserved.

(Dword Count * 4)+3:4
Command Dwords (CDWS): Contains a sequential list of Command Dwords from
the Set Features command  starting with Command Dword 10. The number of
entries in the list is specified by the DWC field. All non-reserved Command Dwords
specified by the Set Features command for the Feature Identifier shall be logged.
The Command Dwords are ordered as defined by the Common Command Format
in Figure 92.
(Memory Buffer Count) +
(Dword Count * 4) + 3:
(Dword Count * 4) + 4
Memory Buffer  (MBUF): Contains the data in the memory buffer for the Set
Features command.
If the Memory Buffer Count field is cleared to 0h, then this field does not exist in
the logged event.
