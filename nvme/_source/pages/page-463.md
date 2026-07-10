---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 463
headings: ["5.3.8.1 Manage Exported NVM Subsystem Management Operations", "5.3.8.1.1 Delete (Management Operation 01h)", "5.3.8.1.2 Change Access Mode (Management Operation 02h)"]
---

# Source page 463

NVM Express® Base Specification, Revision 2.1

441
Figure 521: Manage Exported NVM Subsystem– Command Dword 10
Bits Description
31:16 Reserved
15:08 Management Operation Specific (MOS): If not defined for the management operation specified by the
Select field, this field is reserved.
07:00
Select (SEL): This field selects the type of management operation to perform.
Value Management Operation Reference Section
00h Reserved
01h Delete 5.3.8.1.1
02h Change Access Mode 5.3.8.1.2
03h Grant Host Access 5.3.8.1.3
04h Revoke Host Access 5.3.8.1.4
05h to FFh Reserved

 Manage Exported NVM Subsystem Management Operations
5.3.8.1.1 Delete (Management Operation 01h)
The Delete operation of the Manage Exported NVM Subsystem command is used to delete a specified
Exported NVM Subsystem. The data buffer for the Delete operation of the Manage Exported NVM
Subsystem command contains an NVM Subsystem NQN specifying the Exported NVM Subsystem to be
deleted.
The Management Operation Specific field in Command Dword 10 is reserved and not used by this
operation.
If a Manage Exported NVM Subsystem command is processed that specifies the Delete operation without
specifying an existing Exported NVM Subsystem identified by the NVM Subsystem NQN (SUBNQN) in the
data buffer, then the controller shall abort the command with a status code of Invalid Field in Command.
If there are:
• any active controllers in the specified Exported NVM Subsystem;
• any associations of Exported Namespaces in the specified Exported NVM Subsystem; or
• any Exported Ports exist in the specified Exported NVM Subsystem,
then the controller shall abort the command with a status code of Command Sequence Error.
Upon successful completion of a Manage Exported NVM Subsystem command with a Delete operation the
Exported NVM Subsystem identified shall be deleted.
5.3.8.1.2 Change Access Mode (Management Operation 02h)
The Change Access Mode operation of the Manage Exported NVM Subsystem command is used to change
the access mode of an Exported NVM Subsystem to:
• allow all hosts access to the specified Exported NVM Subsystem, or
• restrict access to specified hosts listed in the Allowed Host List associated to the specified Exported
NVM Subsystem.
The data buffer for the Change Access Mode operation of the Manage Exported NVM Subsystem command
specifies an NVM Subsystem NQN indicating the Exported NVM Subsystem for which the Access Mode is
to be changed. The Management Operation Specific field in Co mmand Dword 10 for the Change Access
Mode operation is shown in Figure 522.
Figure 522: Management Operation Specific: Change Access Mode operation
Bits Description
15:09 Reserved
