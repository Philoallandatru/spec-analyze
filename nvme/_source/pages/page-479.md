---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 479
headings: ["6.5 Property Get Command and Response"]
---

# Source page 479

NVM Express® Base Specification, Revision 2.1

457
The host should not submit commands to an I/O Submission Queue after the submission of a Disconnect
command to that I/O Submission Queue; submitting commands to an I/O Queue after a Disconnect
command is submitted to that I/O Queue results in undefined behavior.
Figure 549: Disconnect Command – Submission Queue Entry
Bytes Description
03:00 Command Dword 0 (CDW0): Refer to Figure 95. Byte 01 is cleared to 0h.
04 Fabrics Command Type (FCTYPE): Set to 08h to specify a Disconnect command.
39:05 Reserved
41:40
Record Format (RECFMT):  Specifies the format of the Disconnect command capsule. The format of
the record specified in this definition shall be 0h. If the NVM subsystem does not support the value
specified, then a status code of Incompatible Format shall be returned.
63:42 Reserved
The Disconnect response provides status for the Disconnect command. The Disconnect response is
defined in Figure 550.
Figure 550: Disconnect Response
Bytes Description
07:00 Reserved
09:08 SQ Head Pointer (SQHD): Indicates the current Submission Queue Head pointer for the associated
Submission Queue.
11:10 Reserved
13:12 Command Identifier (CID): Indicates the identifier of the command that is being completed.
15:14
Status Info (STS): Indicates status for the command.
Bits Description
15:01 Status (STATUS): The Status field for the command. Refer to section 4.2.2.
Refer to Figure 105 for values specific to the Disconnect command.
00 Reserved

6.5 Property Get Command and Response
The Property Get command is used to specify the property value to return to the host (refer to section 3.1.4).
The fields for the Property Get command are defined in Figure 551. If an invalid property or invalid offset is
specified, then a status code of Invalid Field in Command shall be returned.
Figure 551: Property Get Command – Submission Queue Entry
Bytes Description
03:00 Command Dword 0 (CDW0): Refer to Figure 95. Byte 01 is cleared to 0h.
04 Fabrics Command Type (FCTYPE): Set to 04h to specify a Property Get command.
39:05 Reserved
40
Attributes (ATTRIB): Specifies attributes for the Property Get command.
Bits Description
7:3 Reserved
2:0
Property Return Size (PRS): This field specifies the size of the property to return. Valid
values are shown in the table below.
Value Definition
000b 4 bytes
001b 8 bytes
010b to 111b Reserved


43:41 Reserved
47:44 Offset (OFST): Specifies the offset to the property to get. Refer to section 3.1.4.
63:48 Reserved
