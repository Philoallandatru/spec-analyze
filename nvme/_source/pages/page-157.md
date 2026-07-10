---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 157
headings: []
---

# Source page 157

NVM Express® Base Specification, Revision 2.1

135
Figure 92: Common Command Format
Bytes Description
23:16
Metadata Pointer (MPTR): If CDW0.PSDT (refer to Figure 91 is cleared to 00b, then this field shall contain
the address of a contiguous physical buffer of metadata and that address shall be dword aligned (i.e., bits 1:0
cleared to 00b). The controller is not required to check that bits 1:0 are cleared to 00b. The  controller may
report an error of Invalid Field in Command if bits 1:0 are not cleared to 00b. If the controller does not report
an error of Invalid Field in Command, then the controller shall operate as if bits 1:0 are cleared to 00b.
If CDW0.PSDT is set to 01b, then this field shall contain the address of a contiguous physical buffer of
metadata. Refer to the Metadata Buffer Alignment (MBA) bit of the SGLS field in the Identify Controller data
structure for alignment requirements.
If CDW0.PSDT is set to 10b, then this field shall contain the address of an SGL segment that contains exactly
one SGL Descriptor. The address of that SGL segment shall be qword aligned (i.e., bits 2:0 cleared to 000b).
The SGL Descriptor contained in that SGL segment is the first SGL Descriptor of the metadata for the
command. If the SGL Descriptor contained in that SGL segment is an SGL Data Block descriptor, then that
SGL Data Block Descriptor is the only SGL Descriptor and therefore describes the entire metadata data
transfer. Refer to section 4.3.2. The controller is not required to check that bits 2:0 are cleared to 000b. The
controller may report an error of Invalid Field in Command if bits 2:0 are not cleared to 000b. If the controller
does not report an error of Invalid Field in Command, then the controller shall operate as if bits 2:0 are cleared
to 000b.
