---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 472
headings: ["6.2 Authentication Send Command and Response"]
---

# Source page 472

NVM Express® Base Specification, Revision 2.1

450
Figure 541: Authentication Receive Command – Submission Queue Entry
Bytes Description
39:24 SGL Descriptor 1 (SGL1): Refer to Figure 94.
40 Reserved
41 SP Specific 0 (SPSP0):  The value of this field contains bits 07:00 of the Security Protocol Specific
field as defined in SPC-5.
42 SP Specific 1 (SPSP1):  The value of this field contains bits 15:08 of the Security Protocol Specific
field as defined in SPC-5.
43
Security Protocol (SECP):  This field specifies the security protocol as defined in SPC-5. The
controller shall abort the command with Invalid Parameter indicated if a reserved value of the Security
Protocol is specified.
47:44 Allocation Length (AL): The value of this field is specific to the Security Protocol as defined in SPC-
5 where INC_512 is cleared to ‘0’.
63:48 Reserved

Figure 542: Authentication Receive Response
Bytes Description
07:00 Reserved
09:08 SQ Head Pointer (SQHD): Indicates the current Submission Queue Head pointer for the associated
Submission Queue.
11:10 Reserved
13:12 Command Identifier (CID): Indicates the identifier of the command that is being completed.
15:14
Status Info (STS): Indicates status for the command.
Bits Description
15:01 Status (STATUS): The Status field for the command. Refer to section 4.2.3.
00 Reserved

6.2 Authentication Send Command and Response
The Authentication Send command is used to transfer security protocol data to the controller. The data
structure transferred as part of this command contains security protocol specific commands to be performed
by the controller. The data structure may cont ain data or parameters associated with the security protocol
specific commands. Status and data that is to be returned to the host for the security protocol specific
commands submitted by an Authentication Send command are retrieved with the Authentication  Receive
command defined in section 6.1.
The association between an Authentication Send command and subsequent Authentication Receive
commands is Security Protocol field dependent as defined in SPC-5.
Figure 543: Authentication Send Command – Submission Queue Entry
Bytes Description
03:00 Command Dword 0 (CDW0): Refer to Figure 95.
04 Fabrics Command Type (FCTYPE): Set to 05h to specify an Authentication Send command.
23:05 Reserved
39:24 SGL Descriptor 1 (SGL1): Refer to Figure 94.
40 Reserved
41 SP Specific 0 (SPSP0):  The value of this field contains bits 07:00 of the Security Protocol Specific
field as defined in SPC-5.
42 SP Specific 1 (SPSP1):  The value of this field contains bits 15:08 of the Security Protocol Specific
field as defined in SPC-5.
43
Security Protocol (SECP):  This field specifies the security protocol as defined in SPC-5. The
controller shall abort the command with a status code of Invalid Parameter indicated if a reserved
value of the Security Protocol is specified.
