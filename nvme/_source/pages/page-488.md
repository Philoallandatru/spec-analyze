---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 488
headings: ["7.4.1 I/O Management Send Operations", "7.4.1.1 Reclaim Unit Handle Update (Management Operation 01h)"]
---

# Source page 488

NVM Express® Base Specification, Revision 2.1

466
Figure 565: I/O Management Send – Command Dword 10
Bits Description
07:00
Management Operation (MO): This field specifies the management operation to perform.
Value Definition
00h No action
01h
Reclaim Unit Handle Update: Update the reference to the Reclaim Unit specified by
each entry in the Placement Identifier list to reference an empty Reclaim Unit (refer to
section 3.2.4).
FFh Vendor specific
All others Reserved

 I/O Management Send Operations
 Reclaim Unit Handle Update (Management Operation 01h)
The Reclaim Unit Handle Update management operation for the I/O Management Send command provides
a list of Placement Identifiers (refer to Figure 567). The number of Placement Identifiers is defined in the
Management Operation Specific field defined in Figure 566. For each Placement Identifier in the list:
• If the currently referenced Reclaim Unit has been written with user data, then the Placement
Identifier shall be modified to reference a different Reclaim Unit that is empty (refer to section 3.2.4);
and
• If the currently referenced Reclaim Unit has not been written with any user data (i.e., is already
empty), then the Placement Identifier may be modified to reference a different Reclaim Unit that is
empty.
Figure 566: Management Operation Specific – Reclaim Unit Handle Update Operation
Bits Description
15:00
Number of Placement Identifiers (NPID):  Indicates the number of Placement Identifiers that are
specified in the command. This is a 0’s based value.
This field shall not exceed the minimum of:
• the value in the Max Placement Identifiers (MAXPIDS) field of the enabled FDP configuration
(refer to Figure 280); and
• the product of the Number of Reclaim Groups (NRG) field and the Number of Reclaim Unit
Handles (NRUH) field of the enabled FDP configuration (refer to Figure 280);
If a specified Placement Identifier is invalid due to:
• the value of the Reclaim Group Identifier field being greater than or equal to the Number of Reclaim
Groups field of the FDP Configuration Descriptor  (refer to Figure 280) for the Endurance Group
associated with the specified namespace; or
• the specified Placement Handle field being greater than or equal to the Number of Placement
Handles field specified when the namespace was created,
then the controller shall abort the command with a status code of Invalid Field in Command.
If the value represented by the Number of Placement Identifiers (NPID) field is greater than the Max
Placement Identifiers (MAXPIDS) field (refer to Figure 280) in the current FDP configuration, then the
controller shall abort the command with a status code of Invalid Field in Command.
If the command is aborted, then Placement Identifiers may or may not have been updated.
While processing an I/O Management Send command that specifies the Reclaim Unit Handle Update
operation, if the controller processes a write command that utilize s a Placement Identifier specified in the
Placement Identifier List of that I/O Management Send command, then the controller may write the user
data for that write command to the referenced Reclaim Unit:
