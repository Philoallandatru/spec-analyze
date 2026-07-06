---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 203
headings: ["5.1.3 Capacity Management command"]
---

# Source page 203

NVM Express® Base Specification, Revision 2.1

181
 Capacity Management command
Host software uses the Capacity Management command to configure Endurance Groups and NVM Sets in
an NVM subsystem by either selecting one of a set of supported configurations (i.e., Fixed Capacity
Management; refer to section 8.1.4.2) or by specifying the capacity of the Endurance Group or NVM Set to
be created (i.e., Variable Capacity Management; refer to section 8.1.4.3). The Capacity Management
command specifies the operations defined in Figure 156.
The Capacity Management command uses the Command Dword 10, Command Dword 11, and Command
Dword 12 fields. All other command specific fields are reserved.
For requirements to support the operations in Figure 156, refer to section 8.1.4.
Figure 156: Capacity Management – Command Dword 10
Bits Description
31:16 Element Identifier (ELID): This field contains a value specific to the value of the Operation field.1
15:04 Reserved
03:00
Operation (OPER): Specifies the operation to be performed by the controller:
Value Description Element Identifier Field
0h
Select Capacity Configuration:  Endurance
Groups and NVM Sets are configured as indicated
by the Capacity Configuration Descriptor specified
by this command (refer to section 5.1.3.1).
Capacity Configuration Identifier (refer
to Figure 257).
1h
Create Endurance Group:  An Endurance Group
is created (refer to section 8.1.4.3) with the capacity
specified by the Capacity Lower field (refer to
Figure 157) and the Capacity Upper field (refer to
Figure 158).
Domain Identifier (refer to section
3.2.5.3) of the domain in which the
Endurance Group is to be created. A
value of 0h specifies that the controller
selects the domain in which the
Endurance Group is created.
2h
Delete Endurance Group: The Endurance Group
specified by this command is deleted (refer to
section 8.1.4.3). All namespaces and NVM Sets
contained by the Endurance Group are deleted.
Endurance Group Identifier of the
Endurance Group to be deleted.
3h
Create NVM Set: An NVM Set is created (refer to
section 8.1.4.3) in the specified Endurance Group
with the capacity specified by the Capacity Lower
field and the Capacity Upper field.
If the controller does not support NVM Sets, then
this operation is not supported.
Endurance Group Identifier of the
Endurance Group in which the NVM
Set is to be created. A value of 0h
specifies that the controller selects the
Endurance Group in which the NVM
Set is created.
4h
Delete NVM Set:  The NVM Set specified by this
command is deleted (refer to section 8.1.4.3). All
namespaces in the NVM Set are deleted.
If the controller does not support NVM Sets, then
this operation is not supported.
NVM Set identifier of the NVM Set to be
deleted.
5h to Fh Reserved

Notes:
1. If the Element Identifier field specifies a non-zero value which does not correspond to an existing capacity entity,
then the controller shall abort the command with a status code of Invalid Field in Command.

Figure 157: Capacity Management – Command Dword 11
Bits Description
31:00 Capacity Lower (CAPL): This field specifies the least significant 32 bits of the capacity in bytes of the
Endurance Group or NVM Set to be created.
