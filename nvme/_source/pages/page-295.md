---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 295
headings: []
---

# Source page 295

NVM Express® Base Specification, Revision 2.1

273
Figure 280: FDP Configuration Descriptor
Bytes Description
03 Vendor Specific Size (VSS): This field identifies the size in bytes of the Vendor Specific
field.
07:04 Number of Reclaim Groups (NRG):  This field indicates the number of Reclaim Groups in
this FDP configuration. This field shall be a non-zero value.
09:08 Number of Reclaim Unit Handles (NRUH): This field identifies the number of Reclaim Unit
Handle Descriptors in the Reclaim Unit Handle List. This field shall be a non-zero value.
11:10
Max Placement Identifiers (MAXPIDS): This field indicates the maximum value allowed in
the Number of Placement Identifiers (NPID) field (refer to Figure 566) in an I/O Management
Send command for the Reclaim Unit Handle Update Operation (refer to section  7.4.1.1) for
this FDP configuration. This is a 0’s based number.
The value of this field shall be less than the product of the NRG field and the NRUH field.
15:12
Number of Namespaces Supported (NNS): This field indicates the number of namespaces
allowed to be created in the Endurance Group if Flexible Data Placement is enabled utilizing
this FDP configuration.
If the MNAN field is set to a value greater than 0h (refer to  Figure 312), then this field shall
be less than or equal to the value in the MNAN field. If the MNAN field is cleared to 0h, then
this field shall be less than or equal to the value in the NN field (refer to Figure 312).
23:16 Reclaim Unit Nominal Size (RUNS): This field indicates the nominal size, in bytes, of each
Reclaim Unit of non-volatile storage within each Reclaim Group.
27:24
Estimated Reclaim Unit Time Limit (ERUTL): This field indicates the estimated maximum
time in seconds that a Reclaim Unit is allowed to be referenced by a Reclaim Unit Handle
(refer to section 3.2.4 and section 8.1.10) before the controller is allowed to modify the
Reclaim Unit Handle to reference a different Reclaim Unit. If this field is cleared to 0h, then
the Estimated Reclaim Unit Time Limit is not reported.
63:28 Reserved
Reclaim Unit Handle Descriptor List
67:64 Reclaim Unit Handle Descriptor 0: This field contains the Reclaim Unit Handle Descriptor
(refer to Figure 281) for Reclaim Unit Handle 0 (refer to Figure 70).
71:68 Reclaim Unit Handle Descriptor 1: This field contains the Reclaim Unit Handle Descriptor
(refer to Figure 281) for Reclaim Unit Handle 1 (refer to Figure 70), if any.
… …
((NRUH-1)*4)+67:
((NRUH-1)*4)+64
Reclaim Unit Handle Descriptor NRUH -1: This field contains the Reclaim Unit Handle
Descriptor (refer to Figure 281) for Reclaim Unit Handle NRUH -1 (refer to Reclaim Unit
Handle N-1 in Figure 70), if any.
(NRUH*4)+63+VSS:
(NRUH*4)+64 Vendor Specific (VS): This field is valid if the VSS field is greater than 0h.
(NRUH*4)+64+VSS  Pad (PAD):  Padding to align the size of this descriptor to the next 8 -byte boundary, if
necessary. This field shall be cleared to 0h.

Figure 281: Reclaim Unit Handle Descriptor
Bytes Description
00
Reclaim Unit Handle Type (RUHT):  This field indicates the Reclaim Unit Handle type of the
Reclaim Unit Handle.
Value Definition
0h Reserved
1h
Initially Isolated: The user data from a write command that utilizes this type
of Reclaim Unit Handle is originally isolated in the referenced Reclaim Unit
from other user data in write commands that utilize a different Reclaim Unit
Handle in the same Reclaim Group. If the controller moves the user data due
to vendor specific operations (i.e., garbage collection), then the controller is
allowed to move that user data to a Reclaim Unit in the same Reclaim Group
that contains other user data moved by the controller that was writ ten by the
host utilizing any Reclaim Unit Handle of the same type. Refer to section
8.1.10.
