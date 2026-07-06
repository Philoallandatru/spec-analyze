---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 464
headings: ["5.3.8.1.3 Grant Host Access (Management Operation 03h)"]
---

# Source page 464

NVM Express® Base Specification, Revision 2.1

442
Figure 522: Management Operation Specific: Change Access Mode operation
Bits Description
08
Restricted Access (RA): A value of ‘0’ configures this Exported NVM Subsystem for unrestricted access
and may be accessed by any host. A value of ‘1’ configures this Exported NVM Subsystem to restrict access
to Host NQNs present in the Allowed Host List.
The Restricted Access bit defined in Figure 522 specifies the access setting for the Exported NVM
Subsystem:
• A value of ‘0’ in the Restricted Access bit configures this Exported NVM Subsystem with
Unrestricted Access. This value shall enable all hosts to access the specified Exported NVM
Subsystem.
• A value of ‘1’ in the Restricted Access bit shall restrict access of Exported NVM Subsystem to
entries in the Allowed Host List associated to the specified Exported NVM Subsystem. Any
connected host not in the Allowed Host List associated to the specified Exported NVM Subsystem
shall be disconnected from all Exported Namespaces in the Exported NVM Subsystem.
If a Manage Exported NVM Subsystem command is processed that specifies the Change Access Mode
operation without specifying an existing Exported NVM Subsystem in the NVM Subsystem NQN (SUBNQN)
field, then the controller shall abort the command with a status code of Invalid Field in Command.
Upon successful completion of a Manage Exported NVM Subsystem command with a Change Access
Mode operation, the access mode for the specified Exported NVM Subsystem shall be set to the value
specified in the Restricted Access bit in the Management Operation Specific field.
5.3.8.1.3 Grant Host Access (Management Operation 03h)
The Grant Host Access operation of the Manage Exported NVM Subsystem command is used to grant
hosts access to Exported NVM Subsystems by adding the specified hosts to the Allowed Host List
associated to the specified Exported NVM Subsystem. The data pointe r shall point to a Subsystem
Management data structure (refer to Figure 523). The Management Operation Specific field in Command
Dword 10 is reserved and not used by this operation.
The Grant Host Access operation may be used to grant access from a list of hosts (refer to the Host Entry
List and “M” in Figure 523) to a list of Exported NVM Subsystems (refer to the Exported NVM Subsystem
Entry List and “N” in Figure 523) through specified port(s) (refer to Port ID of the Underlying Port in Figure
523) on each of the Exported NVM Subsystems.
