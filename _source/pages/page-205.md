---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 205
headings: ["5.1.3.3 NVM Set Operations"]
---

# Source page 205

NVM Express® Base Specification, Revision 2.1

183
If the Operation field specifies the Create Endurance Group operation and Media Units are supported, then
the controller selects Media Units from the specified domain to be allocated to the Endurance Group.
If the Operation field specifies the Create Endurance Group operation and Media Units are not supported,
then the controller selects NVM capacity from the specified domain to be allocated to the Endurance Group.
If the Operation field specifies the Create Endurance Group operation and the Capacity Lower and Capacity
Upper fields specify a value that requires allocation of NVM capacity that is greater than the value in:
a) the Max Endurance Group Capacity (MEGCAP) field in the Identify Controller data structure;
b) the Unallocated NVM Capacity (UNVMCAP) field in the Identify Controller data structure; or
c) the Max Endurance Group Capacity (MEGCAP) field in the Domain Attributes Entry for the domain
in which the Endurance Group is being created,
then the controller:
a) shall abort the command with Insufficient Capacity status; and
b) if the Error Information log page is supported, shall indicate in the Command Specific Information
field in the Error Information Log Entry data structure (refer to Figure 205) the total amount of NVM
capacity in bytes of the largest Endurance Group that is able to be created.
If the Operation field specifies the Delete Endurance Group operation and the Element Identifier field
specifies a value of 0h or the identifier of an Endurance Group that does not exist, then the controller shall
abort the command with a status code of Invalid Field in Command.
 NVM Set Operations
If the Operation field specifies the Create NVM Set operation, the controller shall select a non -zero NVM
Set Identifier not assigned to an existing NVM Set in the specified Endurance Group (refer to Figure 156)
and indicate that value in the completion queue entry.  If a non -zero unassigned NVM Set Identifier is
unavailable, then the controller shall abort the command with a status code of Identifier Unavailable.
If the Operation field specifies the Create NVM Set operation and the Capacity Lower and Capacity Upper
fields specify a value that requires allocation of NVM capacity that is greater than the value in the
Unallocated Endurance Group Capacity (UEGCAP) fiel d in the Endurance Group Information log page
(refer to Figure 218) for the specified Endurance Group, then the controller:
a) if the Error Information log page is supported, shall indicate in the Command Specific Information
field in the Error Information Log Entry data structure (refer to Figure 205) the total amount of NVM
capacity in bytes of the largest NVM Set that is able to be created; and
b) shall abort the command with Insufficient Capacity status.
If the Operation field specifies the Create NVM Set operation and the Element Identifier field is cleared to
0h, then the controller shall choose an existing Endurance Group in an existing domain in which to create
the NVM Set.
If the Operation field specifies the Create NVM Set operation and Media Units are supported, then the
controller selects Media Units from the Endurance Group to be allocated to the NVM Set.
If the Operation field specifies the Create NVM Set operation and Media Units are not supported, then the
controller selects NVM capacity from the Endurance Group to be allocated to the NVM Set.
If the Operation field specifies the Delete NVM Set operation and the Element Identifier field specifies a
value of 0h or the identifier of an NVM Set that does not exist, then the controller shall abort the command
with a status code of Invalid Field in Command.
If the controller does not support NVM Sets and the Operation field specifies either the Create NVM Set
operation or the Delete NVM Set operation, then the controller shall abort the command with a status code
of Invalid Field in Command.
