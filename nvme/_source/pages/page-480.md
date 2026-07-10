---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 480
headings: ["6.6 Property Set Command and Response"]
---

# Source page 480

NVM Express® Base Specification, Revision 2.1

458
The Property Get response is used to return the value of the property requested to the host. The Property
Get response is defined in Figure 552.
Figure 552: Property Get Response
Bytes Description
07:00
Value (VALUE): Indicates the value returned for the property if the Property Get command is
successful. If the size of the property is four bytes, then the value is specified in bytes 03:00 and bytes
07:04 are reserved.
09:08 SQ Head Pointer (SQHD): Indicates the current Submission Queue Head pointer for the associated
Submission Queue.
11:10 Reserved
13:12 Command Identifier (CID): Indicates the identifier of the command that is being completed.
15:14
Status Info (STS): Indicates status for the command.
Bits Description
15:01 Status (STATUS): The Status field for the command. Refer to section 4.2.2.
00 Reserved

6.6 Property Set Command and Response
The Property Set command is used to set the value of a property (refer to section  3.1.4). The fields for the
Property Set command are defined in Figure 553. If an invalid property or invalid offset is specified, then a
status code of Invalid Field in Command shall be returned.
Figure 553: Property Set Command – Submission Queue Entry
Bytes Description
03:00 Command Dword 0 (CDW0): Refer to Figure 95. Byte 01 is cleared to 0h.
04 Fabrics Command Type (FCTYPE): Cleared to 00h to specify a Property Set command.
39:05 Reserved
40
Attributes (ATTRIB): Specifies attributes for the Property Set command.
Bits Description
7:3 Reserved
2:0
Property Update Size (PUS): This field specifies the size of the property to update. Valid
values are shown in the table below.
Value Definition
000b 4 bytes
001b 8 bytes
010b to 111b Reserved


43:41 Reserved
47:44 Offset (OFST): Specifies the offset to the property to set. Refer to section 3.1.4.
55:48 Value (VALUE): Specifies the value used to update the property. If the size of the property is four
bytes, then the value is specified in bytes 51:48 and bytes 55:52 are reserved.
63:56 Reserved
The Property Set response provides status for the Property Set command. The Property Set response is
defined in Figure 554.
Figure 554: Property Set Response
Bytes Description
07:00 Reserved
09:08 SQ Head Pointer (SQHD): Indicates the current Submission Queue Head pointer for the associated
Submission Queue.
11:10 Reserved
13:12 Command Identifier (CID): Indicates the identifier of the command that is being completed.
