---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 462
headings: ["5.3.7.2 Command Completion", "5.3.8 Manage Exported NVM Subsystem command"]
---

# Source page 462

NVM Express® Base Specification, Revision 2.1

440
Figure 519: Disassociate Namespace Data Structure
Bytes Description
31:00 Exported Namespace ID (ENSID):  A specified Exported NSID associated with the specified Exported
NVM Subsystem
287:32 Exported NVM Subsystem NQN  (ENSNQN): Specifies the Exported NVM Subsystem NQN that is
associated with the specified Exported Namespace ID specified in the ENSID field.
319:288 Reserved
If the Exported Namespace is attached to any controller, then the controller processing the command shall
abort the command with a status code of Command Sequence Error.
If the Disassociate Namespace data structure does not contain an Exported NVM Subsystem NQN
(SUBNQN) for an existing Exported NVM Subsystem associated with the specified Exported Namespace
ID and an Exported NSID associated with the specified Exported NVM Subsystem, then the controller shall
abort the command with a status code of Invalid Field in Command.
Upon successful completion of a Manage Exported Namespace command with a Disassociate Namespace
operation:
• the specified Exported Namespace ID has been deleted; and
• a Namespace Attribute Changed asynchronous event is reported as described in Figure 151.
 Command Completion
Upon completion of the Manage Exported Namespace command, the controller posts a completion queue
entry to the Admin Completion queue indicating the status of the command. Refer to section  8.3.3 for
usages.
 Manage Exported NVM Subsystem command
The Manage Exported NVM Subsystem command is used to configure and manage an Exported NVM
Subsystem.
The Manage Exported NVM Subsystem command uses the Data Pointer and Command Dword 10. All
other command specific fields are reserved.
The Select field defined in Figure 521 determines which management operation is to be performed by this
command. The specified management operation determines the data structure used as part of the
command. The data structure is 4,096 bytes in size . Refer to section 5.3.8.1 for a description of each
management operation.
The Manage Exported NVM Subsystem command shall not be supported by  Exported NVM Subsystems.
Figure 520: Manage Exported NVM Subsystem – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer Figure 92 for the definition of
this field.
