---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 380
headings: []
---

# Source page 380

NVM Express® Base Specification, Revision 2.1

358
If the host specifies an offset (i.e., the CSLO field and the CSUO field) that is greater than the size of the
Controller State data, then the controller shall abort the command with a status code of Invalid Field in
Command.
Figure 358: Controller State Data Structure
Bytes Description
Header
01:00 Version (VER): This field indicates the version of this data structure. This field shall be
cleared to 0h.
02
Controller State Attributes (CSATTR): This field specifies attributes associated with
the controller state.

Bits Description
07:01 Reserved
00
Controller Suspended (CP): If this bit is set to ‘1’, then the controller
associated with this data structure was suspended for the entire duration of
the processing of the Migration Receive command that reported in contents
this data structure. If this bit is cleared to ‘0’, then the controller associated
with this data structure was not suspended for the entire duration of the
processing of the Migration Receive command that reported the contents
in this data structure.
15:03 Reserved
31:16 NVMe Controller State Size (NVMECSS): This field indicates the number of dwords
in the NVMe Controller State field.
47:32 Vendor Specific Size (VSS): This field indicates the number of dwords in the Vendor
Specific Data field.
Controller State Data
(NVMECSS*4)+47:48
NVMe Controller State (NVMECS): If the NVMECSS field is non-zero, then this field
contains the controller state data as defined in  Figure 359. If the NVMECSS field is
cleared to 0h, then this field does not exist in the data structure.
((VSS+NVMECSS)*4)+47:
(NVMECSS*4)+48
Vendor Specific Data (VSD): If the VSS field is non -zero, then this field contains
vendor specific data. If the VSS field is cleared to 0h, then this field does not exist in
the data structure.
If the CSVI field (refer to  Figure 354) specifies a non -zero index that is not defined by the Supported
Controller State Formats data structure (refer to Figure 329), then the controller shall abort the command
with a status code of Invalid Field in Command.
If the CSVI field (refer to Figure 354) is cleared to 0h and the NVMe Controller State Size field is non-zero,
then the controller shall abort the command with a status code of Invalid Field in Command.
If the CSUUIDI field (refer to Figure 354) specifies a non -zero index that is not defined by the Supported
Controller State Formats data structure, then the controller shall abort the command with a status code of
Invalid Field in Command.
If the NVMe Controller State field exists (i.e., the NVMECSS field is non -zero) and any I/O Submission
Queue or I/O Completion Queue exists in the controller specified by the Controller Identifier (CNTLID) field,
then the controller shall abort the command with a status code of Invalid Field in Command.
Figure 359 defines the NVMe Controller State data structure that identifies the state of the all the I/O
Submission Queues and I/O Completion Queues for a controller . Any controller state defined by this
specification that is not included in the NVMe Controller State data structure is either included in the Vendor
Specific Data field in the Controller State data structure (refer to Figure 358) or is obtained in a vendor
specific manner.
