---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 85
headings: ["3.1.4.7 Offset 20h: NSSR – NVM Subsystem Reset", "3.1.4.8 Offset 24h: AQA – Admin Queue Attributes", "3.1.4.9 Offset 28h: ASQ – Admin Submission Queue Base Address"]
---

# Source page 85

NVM Express® Base Specification, Revision 2.1

63
 Offset 20h: NSSR – NVM Subsystem Reset
This optional property provides host software with the capability to initiate an NVM Subsystem Reset.
Support for this property is indicated by the state of the NVM Subsystem Reset Supported (CAP.NSSRS)
field. If the property is not supported, then the address range occupied by the property is reserved. Refer
to section 3.7.1.
Figure 43: Offset 20h: NSSR – NVM Subsystem Reset
Bits Type Reset Description
31:00 RW 0h
NVM Subsystem Reset Control (NSSRC): A write of the value 4E564D65h ("NVMe")
to this field initiates an NVM Subsystem Reset. A write of any other value has no
functional effect on the operation of the NVM subsystem. This field shall return the
value 0h when read.
 Offset 24h: AQA – Admin Queue Attributes
This property defines the attributes for the Admin Submission Queue and Admin Completion Queue. The
Queue Identifier for the Admin Submission Queue and Admin Completion Queue is 0h. The Admin
Submission Queue’s priority is determined by the arbitration mechanism selecte d, refer to section 3.4.4.
The Admin Submission Queue and Admin Completion Queue are required to be in physically contiguous
memory.
This property shall not be reset by Controller Reset.
Note: It is recommended that the host use UEFI during boot operations. In low memory environments (e.g.,
Option ROMs in legacy BIOS environments) there may not be sufficient available memory to allocate the
necessary Submission and Completion Queues. In these types of conditions, low memory operation of the
controller is vendor specific.
Figure 44: Offset 24h: AQA – Admin Queue Attributes
Bits Type Reset Description
31:28 RO 0h Reserved
27:16 RW 0h
Admin Completion Queue Size (ACQS): Defines the size of the Admin Completion
Queue in entries. Refer to section 3.3.3.1. Enabling a controller while this field is cleared
to 0h produces undefined results. The minimum size of the Admin Completion Queue is
two entries. The maximum size of the Admin Completion Queue is 4,096 entries. This is
a 0’s based value.
15:12 RO 0h Reserved
11:00 RW 0h
Admin Submission Queue Size (ASQS):  Defines the size of the Admin Submission
Queue in entries. Refer to section 3.3.3.1. Enabling a controller while this field is cleared
to 0h produces undefined results. The minimum size of the Admin Submission Queue is
two entries. The maximum size of the Admin Submission Queue is 4,096 entries. This is
a 0’s based value.
 Offset 28h: ASQ – Admin Submission Queue Base Address
This property defines the base memory address of the Admin Submission Queue.
This property shall not be reset by Controller Reset.
Figure 45: Offset 28h: ASQ – Admin Submission Queue Base Address
Bits Type Reset Description
63:12 RW Impl
Spec
Admin Submission Queue Base (ASQB): This field specifies the 52 most significant
bits of the 64-bit physical address for the Admin Submission Queue. This address shall
be memory page aligned (based on the value in CC.MPS). All Admin commands,
including creation of I/O Submission Queues and I /O Completions Queues shall be
submitted to this queue. For the definition of Submission Queues, refer to section 4.1.
