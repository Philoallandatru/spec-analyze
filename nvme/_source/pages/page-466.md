---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 466
headings: ["5.3.8.1.4 Revoke Host Access (Management Operation 04h)", "5.3.8.2 Command Completion", "5.3.9 Manage Exported Port command"]
---

# Source page 466

NVM Express® Base Specification, Revision 2.1

444
Figure 526: Modify Host Access – Command Operation Specific Status Values
Value Definition
35h Invalid Host
36h Invalid NVM Subsystem
5.3.8.1.4 Revoke Host Access (Management Operation 04h)
The Revoke Host Access operation of the Manage Exported NVM Subsystem command is used to revoke
access to the specified Exported NVM Subsystems from a list of specified hosts. The data pointer shall
point to a Subsystem Management data structure (refer to Figure 523). The Management Operation
Specific field in Command Dword 10 is reserved and not used by this operation.
The Revoke Host Access operation may be used to revoke 1 to N Hosts access from:
• a specified Exported NVM Subsystem through 1 to M specified ports; or
• N different Exported NVM Subsystems through a specified port on each Exported NVM Subsystem.
Connected Hosts that have access revoked from a specified Exported NVM Subsystem on a specified
Exported Port shall be disconnected from the specified Exported NVM Subsystem(s) on the specified
Exported port.
Command specific status values associated with the Revoke Host Access operation of the Manage
Exported NVM Subsystem command are defined in Figure 526. For failures, the byte offset of a failing entry
shall be reported in the Command Specific Information field of the Error Information Log Entry.
 Command Completion
Upon completion of the Manage Exported NVM Subsystem command, the controller posts a completion
queue entry to the Admin Completion Queue indicating the status for the command. Reference sections
5.3.8.1.1 through 5.3.8.1.4 for operation specific command completion details. Refer to section  8.3.3 for
usages.
 Manage Exported Port command
The Manage Exported Port command is used to manage associations of Exported Ports with Exported
NVM Subsystems. The Manage Exported Port command uses the Data Pointer and Command Dword 10.
All other command specific fields are reserved.
The Select field defined in Figure 528 determines which management operation is to be performed in this
command. The specified management operation determines the data structure used as part of the
command. The data structure is 4,096 bytes in size.
The Manage Exported Port command shall not be supported by Exported NVM Subsystems.
Figure 527: Manage Exported Port – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.
