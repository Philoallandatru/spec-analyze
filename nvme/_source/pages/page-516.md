---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 516
headings: ["8.1.5 Command and Feature Lockdown"]
---

# Source page 516

NVM Express® Base Specification, Revision 2.1

494
 Command and Feature Lockdown
The Command and Feature Lockdown capability is used to prohibit the execution of commands submitted
to NVM Express controllers and/or Management Endpoints in an NVM subsystem. Within this feature,
commands and Feature Identifiers are defined with the following scopes:
• Admin Command Set commands defined by the Opcode field;
• Set Features command features defined by the Feature Identifier field;
• Management Interface Command Set commands defined by the Opcode field (refer to the NVM
Express Management Interface Specification); and
• PCIe Command Set commands defined by the Opcode field (refer to the NVM Express
Management Interface Specification).
Admin Command Set commands and Feature Identifiers are defined to be prohibitable by this feature,
however it is vendor specific which of the Command Set commands and Feature Identifiers are prohibitable
from execution, including the Lockdown command itself.
The prohibition of commands or Feature Identifiers on an interface is specified in the Interface field of the
Lockdown command (refer to section 5.1.15). The prohibition applies to all applicable:
• NVM Express controllers; and
• Management Endpoints,
in the NVM subsystem, as specified in the Interface field.
The Lockdown command is used to specify commands that are prohibited from execution (i.e., locked
down) and may be used further to then again allow that command to be executed.
The prohibiting of execution of a command as part of this feature shall persist until:
a) power cycle of the NVM subsystem; or
b) further being allowed by a follow-on Lockdown command.
If a prohibited Admin Command Set command or Feature Identifier is processed by a controller in the NVM
subsystem, then that command shall be aborted with a status code of Command Prohibited by Command
and Feature Lockdown.
If a prohibited Management Interface Command Set command or PCIe Command Set command is
processed by a management endpoint in the NVM subsystem, then that command shall be aborted and
send a Response Message with an Access Denied Error Response (refer to the NVM Express Management
Interface Specification).
The prohibition or allowance of the execution of a command is based on the  interface on  which the
command is received (i.e., the Admin Submission Queue, or an out -of-band Management Endpoint). For
example, a command is able to be prohibited if received on an Admin Submission Queue but allowed if
received on an out -of-band Management Endpoint, if supported. The Interface field in the Lockdown
command (refer to 5.1.15) is used to specify this behavior.
A host may use the Command and Feature Lockdown log page (refer to section 5.1.12.1.20) to determine
the commands and Feature Identifiers that are allowed to be prohibited from execution. A Get Log Page
command specifying the Command and Feature Lockdown log page returns a list of Admin command
opcodes or Feature Identifiers depending on the scope specified in the Get Log Page command. The
returned list of opcodes or Feature Identifiers are the opcodes or Feature Identifiers that are:
a) supported as being prohibitable from execution using the Lockdown command;
b) currently prohibited from execution if received on an Admin Submission Queue; or
c) currently prohibited from execution if received out-of-band on a Management Endpoint.
If the Command and Feature Lockdown capability is supported (i.e., the CFLS bit in the OACS field in
Figure 312 is set to ‘1’), then the controller shall support the Lockdown command and the Command and
Feature Lockdown log page.
