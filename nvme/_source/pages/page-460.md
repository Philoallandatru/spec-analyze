---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 460
headings: ["5.3.6.1 Command Completion", "5.3.7 Manage Exported Namespace command", "5.3.7.1 Manage Exported Namespace Management Operations", "5.3.7.1.1 Associate Namespace (Management Operation 01h)"]
---

# Source page 460

NVM Express® Base Specification, Revision 2.1

438
 Command Completion
Upon completion of the Fabric Zoning Send (FZS) command, the controller posts a completion queue entry
to the Admin Completion Queue indicating the status of the command. Command specific status values for
the FZS command are defined in Figure 515.
Figure 515: Fabric Zoning Send (FZS) – Command Specific Status Values
Value Definition
30h Zoning Data Structure Locked: The requested Zoning data structure is locked on the CDC.
31h Zoning Data Structure Not Found: The requested Zoning data structure does not exist on the CDC.
33h Requested Function Disabled: Fabric Zoning is not enabled on the CDC.
34h ZoneGroup Originator Invalid: The DDC is not allowed to access the specified ZoneGroup.
 Manage Exported Namespace command
The Manage Exported Namespace command is used to manage associations of Exported Namespaces
with Exported NVM Subsystems. The Manage Exported Namespace command uses the Data Pointer and
Command Dword 10. All other command specific fields are reserved.
The Select field defined in Figure 517determines which management operation is to be performed in this
command. The specified management operation determines the data structure used as part of the
command. The data structure is 4,096 bytes in size (refer to subsections below for description of operation
specific data structures).
The Manage Exported Namespace command shall not be supported by Exported NVM Subsystems.
Figure 516: Manage Exported Namespace – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 517: Manage Exported Namespace – Command Dword 10
Bits Description
31:08 Reserved
07:00
Select (SEL): This field selects the type of management operation to perform.
Value Management Operation Reference Section
00h Reserved
01h Associate Namespace 5.3.7.1.1
02h Disassociate Namespace 5.3.7.1.2
03h to FFh Reserved


 Manage Exported Namespace Management Operations
5.3.7.1.1 Associate Namespace (Management Operation 01h)
The Associate Namespace operation of the Manage Exported Namespace command is used to create an
association between an Exported Namespace ID (ENSID) and an Underlying Namespace; this command
also associates the ENSID with an Exported NVM Subsystem identified by the Exported NVM Subsystem
NQN.
The Underlying Namespace to be associated with the Exported Namespace ID is returned in the Underlying
Namespace List as an Underlying Namespace List Entry (refer to Figure 334) and identified with an
Underlying Controller ID, Underlying NVM Subsystem, and Underlying Namespace ID combination. That
information is specified in an Associate Namespace data structure (refer to Figure 518).
The Manage Exported Namespace command with an Associate Namespace operation does not attach the
Exported Namespace to a controller. The Exported Namespace becomes an attached namespace when
