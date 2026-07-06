---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 204
headings: ["5.1.3.1 Media Unit Configuration Selection", "5.1.3.2 Endurance Group Operations"]
---

# Source page 204

NVM Express® Base Specification, Revision 2.1

182
Figure 158: Capacity Management – Command Dword 12
Bits Description
31:00 Capacity Upper (CAPU): This field specifies the most significant 32 bits of the capacity in bytes of the
Endurance Group or NVM Set to be created.
If the Operation field specifies the Create Endurance Group operation or the Create NVM Set operation,
then the Capacity Upper and Capacity Lower fields specify the capacity of the Endurance Group or NVM
Set to be created. If the Operation field specifies any other operation, then the Capacity Upper field and the
Capacity Lower field are reserved.
 Media Unit Configuration Selection
If:
a) the Operation field specifies the Select Capacity Configuration operation;
b) the Element Identifier field specifies a Capacity Configuration Descriptor in the Supported Capacity
Configuration List; and
c) the Selected Configuration field of the Media Unit Status log page (refer to Figure 254) is cleared
to 0h,
then the controller shall perform all of the following actions in sequence:
1) Create an Endurance Group for each Endurance Group Configuration Descriptor in the selected
Capacity Configuration Descriptor.
2) Create an NVM Set for each NVM Set Identifier specified in each Endurance Group Configuration
Descriptor, if any.
If the Operation field specifies the Select Capacity Configuration operation and the Element Identifier field
is cleared to ‘0’, then the controller shall clear the configuration by performing all of the following actions in
sequence:
1) Delete all namespaces in the domain that contains the controller processing the command, as
described in section 8.1.15.
2) Delete all NVM Sets in the domain that contains the controller processing the command , if any.
3) Delete all Endurance Groups in the domain that contains the controller processing the command.
4) Clear the Selected Configuration field to 0h in the Media Unit Status log page.
If the Operation field specifies the Select Capacity Configuration operation and the Element Identifier field
specifies the value reported in the Selected Configuration field of the Media Unit Status log page (i.e., the
currently-selected configuration), t hen the controller shall complete the command without error and shall
make no change to the capacity configuration.
If the Operation field specifies the Select Capacity Configuration operation and:
a) the Element Identifier field does not specify either a value of 0h or the Capacity Configuration
Identifier of a Capacity Configuration Descriptor in the Capacity Configuration List; or
b) the Selected Configuration field of the Media Unit Status log page (refer to Figure 254) is not cleared
to 0h,
then the controller shall abort the command with a status code of Invalid Field in Command and no changes
shall be made to the configuration of any Media Unit.
 Endurance Group Operations
If the Operation field specifies the Create Endurance Group operation, the controller shall select a non-zero
Endurance Group Identifier not assigned to an existing Endurance Group in the specified domain (refer to
Figure 156) and indicate that value in Dword 0 of the completion queue entry (refer to Figure 160). If a non-
zero unassigned Endurance Group Identi fier is unavailable, then the controller shall abort the command
with a status code of Identifier Unavailable.
