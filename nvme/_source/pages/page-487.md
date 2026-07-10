---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 487
headings: ["7.3.2 Command Completion", "7.4 I/O Management Send command"]
---

# Source page 487

NVM Express® Base Specification, Revision 2.1

465
If the NSID field is set to 0h or FFFFFFFFh, then the controller shall abort the command with a status code
of Invalid Namespace or Format.
If the host reads beyond the size of the Reclaim Unit Handle Status data structure (refer to Figure 563),
zeroes are returned.
Figure 563: Reclaim Unit Handle Status
Bytes Description
Header
13:00 Reserved
15:14
Number of Reclaim Unit Handle Status Descriptors (NRUHSD): This field indicates the
number of Reclaim Unit Handle Status Descriptors in the Reclaim Unit Handle Status
Descriptor list.
Reclaim Unit Handle Status Descriptor List
47:16 Reclaim Unit Handle Status Descriptor 1: The first Reclaim Unit Handle Status
Descriptor (refer to the applicable I/O Command Set specification).
79:48 Reclaim Unit Handle Status Descriptor 2: The second Reclaim Unit Handle Status
Descriptor (refer to the applicable I/O Command Set specification), if any.
… …
(NRUHSD *32)+15:
(NRUHSD *32)–16
Reclaim Unit Handle Status Descriptor NRUHSD: The last Reclaim Unit Handle Status
Descriptor (refer to the applicable I/O Command Set specification), if any.
The Reclaim Unit Handle Status Descriptors are defined in the I/O Command Set specifications, if
supported.
 Command Completion
When the command is completed with success or failure, the controller shall post a completion queue entry
to the associated I/O Completion Queue indicating the status for the command.
7.4 I/O Management Send command
The I/O Management Send command is used to manage I/O and the behavior of the command is dependent
on the specified operation as defined in the Management Operation field in Figure 565.
The command uses the Data Pointer and Command Dword 10 field. All other command specific fields are
reserved. If the command uses PRPs for the data transfer, then the PRP Entry 1 and PRP Entry 2 fields
are used. If the command uses SGLs for the data transf er, then the SGL Entry 1 field is used. All other
command specific fields are reserved.
Figure 564: I/O Management Send – Data Pointer
Bits Description
127:00 Data Pointer (DPTR):  This field specifies the location of a data buffer where data is transferred from.
Refer to Figure 92 for the definition of this field.

Figure 565: I/O Management Send – Command Dword 10
Bits Description
31:16
Management Operation Specific (MOS):  The definition of this field is specific to the value  of the MO
field. If this field is not defined for the management operation specified by the MO field, then this field is
reserved.
15:08 Reserved
