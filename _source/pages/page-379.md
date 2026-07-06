---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 379
headings: []
---

# Source page 379

NVM Express® Base Specification, Revision 2.1

357
Figure 353: Set Controller State – Management Operation Specific Field
Bits Description
15:02 Reserved
01:00
Sequence Indicator (SEQIND): This field identified the sequences of this Migration Send command in
relation to other Migration Send commands.
Value Definition
00b
This command is not the first or last of a sequence of two or more Migration Send commands
with this management operation used to transfer the controller state from the host to the
controller.
01b This command is the first of a sequence of two or more Migration Send commands with this
management operation used to transfer the controller state from the host to the controller.
10b
This Migration Send command is the last command of a sequence of two or more Migration
Send commands with this management operation used to transfer the controller state from
the host to the controller.
11b
This Migration Send command is the only command and contains the entire controller state
for this management operation used to transfer the controller state from the host to the
controller.


Figure 354: Set Controller State – Command Dword 11
Bits Description
31:24
Controller State UUID Index (CSUUIDI): A non-zero value in this field specifies the index to a specific entry
in the Vendor Specific Controller State UUID Supported list of the Supported Controller State Formats data
structure (refer to section 5.1.13.2.21). The contents of that entry specify the format of the Vendor Specific
field in the specified Controller State data structure.
If this field is cleared to 0h, then no Vendor Specific field is contained in the Controller State data structure.
23:16
Controller State Version Index (CSVI): A non-zero value in this field specifies the index to a specific entry
in the NVMe Controller State Version list of the Supported Controller State Formats data structure (refer to
section 5.1.13.2.21). The contents of that entry specify the format of the Controller State field in the specified
Controller State data structure.
If this field is cleared to 0h, then no NVMe Controller State field is specified in the Controller State data
structure.
15:00 Controller Identifier (CNTLID): This field specifies the identifier of the controller whose state is being set.

Figure 355: Set Controller State – Command Dword 12
Bits Description
31:00
Controller State Offset Lower (CSOL): This field specifies the least -significant 32-bits of starting offset
within the Controller State data structure that the host is setting.
The offset is required to be dword aligned and if bits 1:0 are not cleared to 00b, then the controller shall
abort the command with a status code of Invalid Field in Command.

Figure 356: Set Controller State – Command Dword 13
Bits Description
31:00 Controller State Offset Upper (CSOU): This field specifies the most -significant 32-bits of starting offset
within the Controller State data that the host is setting. Refer to the CSOL field definition in Figure 355.

Figure 357: Set Controller State – Command Dword 15
Bits Description
31:00
Number of Dwords (NUMD): This field specifies the number of dwords being transferred.
This field is only allowed to be cleared to the value of 0h if the Sequence Indicator field is set to 10b.
