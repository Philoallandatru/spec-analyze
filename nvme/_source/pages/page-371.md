---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 371
headings: ["5.1.15.1 Command Completion"]
---

# Source page 371

NVM Express® Base Specification, Revision 2.1

349
Figure 337: Lockdown – Command Dword 10
Bits Description
04
Prohibit (PRHBT): This bit specifies whether to prohibit or allow the command opcode or Set Features
Feature Identifier specified by this command. If  this bit is  set to ‘1’, then this command prohibits the
execution of the command based on other fields specified in Dword 10. If this bit is cleared to ‘0’, then this
command allows the execution of the command based on other fields specified in Dword 10.
03:00
Scope (SCP): This field specifies the contents of the Opcode or Feature Identifier field.
Value Opcode or Feature Identifier Definition
0h Admin command opcode
1h Reserved
2h A Set Features Feature Identifier
3h Management Interface Command Set opcode (refer to the NVM Express Management
Interface Specification)
4h PCIe Command Set opcode (refer to the NVM Express Management Interface Specification)
5h-Fh Reserved

If the controller supports selection of a UUID:
a) by the Lockdown command; and
b) by the Set Features command (refer to  section 5.1.25 and section 8.1.28) and for the vendor
specific Feature Identifier specified by the Opcode or Feature Identifier field, if the Scope field is
set to 2h,
then Command Dword 14 (refer to Figure 338) is used to specify a UUID Index value.
If the controller does not support selection of a UUID:
a) by the Lockdown command;
b) by the Set Features command; or
c) for the vendor specific feature identifier specified by the Opcode or Feature Identifier field, if the
Scope field is set to 2h,
then Command Dword 14 does not specify a UUID Index value . If the Scope field is not set to 2h, then
UUID Index field is ignored.
Figure 338: Lockdown – Command Dword 14
Bits Description
31:07 Reserved
06:00 UUID Index (UIDX): Refer to Figure 658.
If a  controller processes this command specifying a command opcode or Feature Identifier that is not
supported as being prohibitable, then the command shall be aborted with a status code of Prohibition of
Command Execution Not Supported.
If a controller processes this command with the Interface field set to 01b or 10b and the NVM subsystem
does not contain a Management Endpoint, then the command shall be aborted with a status code of Invalid
Field in Command.
If a controller processes this command with the Interface field set to 00b or 01b and the Scope field is set
to 4h, then the command shall be aborted with a status code of Invalid Field in Command.
It is not an error to attempt to prohibit a command or Feature Identifier that is already prohibited from
execution or allow a command or Feature Identifier that is already allowed to be executed .
 Command Completion
Upon completion of the Lockdown command, the controller posts a completion queue entry to the Admin
Completion Queue.
