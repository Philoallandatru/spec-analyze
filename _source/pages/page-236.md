---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 236
headings: ["5.1.12.1.6 Commands Supported and Effects (Log Page Identifier 05h)"]
---

# Source page 236

NVM Express® Base Specification, Revision 2.1

214
5.1.12.1.6 Commands Supported and Effects (Log Page Identifier 05h)
This log page is used to describe the commands that the controller supports and the effects of those
commands on the state of the NVM subsystem.  The log page is 4 ,096 bytes in size. There is one
Commands Supported and Effects data structure per Admin command opcode value. For controllers that
implement I/O Queues, there is one Commands Supported and  Effects data structure per I/O command
opcode value for a specified I/O Command Set based on:
a) the I/O Command Set selected in CC.CSS, if CC.CSS is not set to 110b; and
b) the Command Set Identifier field in CDW 14, if CC.CSS is set to 110b.
Figure 209: Commands Supported and Effects Log Page
Bytes Description
03:00 Admin Command Supported 0 (ACS0): Contains the Command s Supported and  Effects data
structure (refer to Figure 210) for the Admin command with an opcode value of 0h.
07:04 Admin Command Supported 1 (ACS1): Contains the Command s Supported and Effects data
structure (refer to Figure 210) for the Admin command with an opcode value of 1h.
… …
1019:1016 Admin Command Supported 254 (ACS254): Contains the Commands Supported and Effects data
structure (refer to Figure 210) for the Admin command with an opcode value of 254.
1023:1020 Admin Command Supported 255 (ACS255): Contains the Commands Supported and Effects data
structure (refer to Figure 210) for the Admin command with an opcode value of 255.
1027:1024 I/O Command Supported 0 (IOCS0): Contains the Command s Supported and  Effects data
structure (refer to Figure 210) for the I/O command with an opcode value of 0h.
1031:1028 I/O Command Supported 1 (IOCS1): Contains the Command s Supported and  Effects data
structure (refer to Figure 210) for the I/O command with an opcode value of 1h.
… …
2043:2040 I/O Command Supported 254 (IOCS254): Contains the Command s Supported and  Effects data
structure (refer to Figure 210) for the I/O command with an opcode value of 254.
2047:2044 I/O Command Supported 255 (IOCS255): Contains the Command s Supported and  Effects data
structure (refer to Figure 210) for the I/O command with an opcode value of 255.
4095:2048 Reserved
The Commands Supported and Effects data structure describes the overall possible effect of a command,
including any optional features of the command.
Host software may take command effects into account when determining how to submit commands and
actions to take after the command is complete. It is recommended that if a command may change a
particular capability that host software re -enumerate and/or re-initialize the associated capability after the
command is complete. For example, if a namespace capability change may occur, then host software is
recommended to pause the use of the associated namespace, submit the command that may cause a
namespace capability change and wait for its completion, and then re-issue the Identify command.
If the namespace is attached to multiple controllers, the host(s) associated with those controllers should
coordinate their commands to meet the Command Submission and Execution requirements (refer to Figure
210). The details of this coordination are outside the scope of this specification.
