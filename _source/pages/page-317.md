---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 317
headings: ["5.1.12.4 Command Completion", "5.1.13 Identify command", "5.1.13.1 Identify command overview"]
---

# Source page 317

NVM Express® Base Specification, Revision 2.1

295
Figure 304: Pull Model DDC Request Log Page
Bytes Description
Header
00
Operation Request Identifier (ORI): This field indicates the operation that a pull model DDC is
requesting a CDC to perform. Defined values are:
Value Definition
0 Reserved
1 Get Active ZoneGroup (GAZ) operation
2 Add or Replace Active ZoneGroup (AAZ) operation
3 Remove Active ZoneGroup (RAZ) operation
4 Discovery Log Page Request operation
All others Reserved

03:01 Reserved
07:04 Total Pull Model DDC Request Log Page Length (TPDRPL): This field indicates the length in
bytes of the entire Pull Model DDC Request log page.
TPDRPL-1:08 Operation Specific Parameters (OSP)
 Command Completion
Upon completion of the Get Log Page command, the controller posts a completion queue entry to the Admin
Completion Queue. Get Log Page command specific status values are defined in Figure 305.
Figure 305: Get Log Page – Command Specific Status Values
Value Description
9h
Invalid Log Page: The log page indicated is invalid  or not supported. This error condition is also
returned if a reserved log page is requested.  Controllers compliant with NVM Express Base
Specification revision 2.0 and earlier may return Invalid Field in Command for this condition.
29h I/O Command Set Not Supported: The specified I/O Command Set is not supported by the
controller.
 Identify command
 Identify command overview
The Identify command returns a data buffer that describes information about the NVM subsystem, the
domain, the controller or the namespace(s). The data structure is 4,096 bytes in size.
The Identify command uses the Data Pointer, Command Dword 10, Command Dword 11, and Command
Dword 14 fields. All other command specific fields are reserved.
Figure 306: Identify – Data Pointer
Bits Description
127:00
Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field. If using PRPs, this field shall not be a pointer to a PRP List as the data buffer may not cross
more than one page boundary.
