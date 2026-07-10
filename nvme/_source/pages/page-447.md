---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 447
headings: ["5.2.6.1 Command Completion"]
---

# Source page 447

NVM Express® Base Specification, Revision 2.1

425
Figure 489: Virtualization Management – Command Dword 10
Bits Description
10:08
Resource Type (RT): This field indicates the type of controller resource to be modified.
Value Definition
000b VQ Resources
001b VI Resources
010b to 111b Reserved

07:04 Reserved
03:00
Action (ACT): This field indicates the operation for the command to perform as described below.
Value Definition
0h Reserved
1h
Primary Controller Flexible Allocation:  Set the number of Flexible Resources
allocated to this primary controller following the next Controller Level Reset other than a
Controller Reset. If the Controller Identifier field does not correspond to this primary
controller, then a status code of Inv alid Controller Identifier is returned. This value is
persistent across power cycles and resets.
2h to 6h Reserved
7h
Secondary Controller Offline: Place the secondary controller in the Offline state and
remove all Flexible Resources. If the Controller Identifier field does not correspond to a
secondary controller associated with this primary controller, then a status code of Invalid
Controller Identifier is returned.
It is not an error to request a secondary controller be placed in the offline state if that
secondary controller is already in the offline state.
8h
Secondary Controller Assign: Assign the number of controller resources specified in
Number of Controller Resources to the secondary controller. If the Controller Identifier
field does not correspond to a secondary controller associated with this primary
controller, then an error of I nvalid Controller Identifier is returned. If the secondary
controller is not in the Offline state, then a status code of Invalid Secondary Controller
State is returned.
9h
Secondary Controller Online: Place the secondary controller in the Online state. If the
Controller Identifier field does not correspond to a secondary controller associated with
this primary controller, then an error of Invalid Controller Identifier is returned. If the
secondary controller is not configured appropriately (refer to section 8.2.6) or the primary
controller is not enabled, then a status code of Invalid Secondary Controller State is
returned.
It is not an error to request a secondary controller be placed in the online state if that
secondary controller is already in the online state.
Ah to Fh Reserved


Figure 490: Virtualization Management – Command Dword 11
Bits Description
31:16 Reserved
15:00 Number of Controller Resources (NR):  This field indicates a number of controller resources to allocate
or assign.
 Command Completion
Command specific status values associated with the Virtualization management command are defined in
Figure 491.
Figure 491: Virtualization Management – Command Specific Status Values
Value Definition
1Fh Invalid Controller Identifier: An invalid Controller Identifier was specified.
