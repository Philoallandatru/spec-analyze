---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 86
headings: ["3.1.4.10 Offset 30h: ACQ – Admin Completion Queue Base Address", "3.1.4.11 Offset 38h: CMBLOC – Controller Memory Buffer Location"]
---

# Source page 86

NVM Express® Base Specification, Revision 2.1

64
Figure 45: Offset 28h: ASQ – Admin Submission Queue Base Address
Bits Type Reset Description
11:00 RO 0h Reserved
 Offset 30h: ACQ – Admin Completion Queue Base Address
This property defines the base memory address of the Admin Completion Queue.
This property shall not be reset by Controller Reset.
Figure 46: Offset 30h: ACQ – Admin Completion Queue Base Address
Bits Type Reset Description
63:12 RW Impl
Spec
Admin Completion Queue Base (ACQB): This field specifies the 52 most significant
bits of the 64-bit physical address for the Admin Completion Queue. This address shall
be memory page aligned (based on the value in CC.MPS). All completion queue entries
for the commands submitted to the Admin Submission Queue shall be posted to this
Completion Queue. This queue is always associated with interrupt vector 0. For the
definition of Completion Queues, refer to section 4.1.
11:00 RO 0h Reserved
 Offset 38h: CMBLOC – Controller Memory Buffer Location
This optional property defines the location of the Controller Memory Buffer (refer to section 8.2.1). If the
controller does not support the Controller Memory Buffer (CAP.CMBS), this property is reserved. If the
controller supports the Controller Memory Buffer and CMBMSC.CRE is cleared to ‘0’, this property shall be
cleared to 0h.
Figure 47: Offset 38h: CMBLOC – Controller Memory Buffer Location
Bits Type Reset Description
31:12 RO Impl
Spec
Offset (OFST): Indicates the offset of the Controller Memory Buffer in multiples of the
Size Unit specified in CMBSZ.
11:09 RO 000b Reserved
08 RO Impl
Spec
CMB Queue Dword Alignment (CQDA): If this bit is set to ‘1’, CDW11.PC is set to ‘1’;
and the address pointer specifies Controller Memory Buffer, then the address pointer in
a Create I/O Submission Queue command (refer to Figure 477) or a Create I/O
Completion Queue command (refer to Figure 473) shall be Dword aligned.
If this bit is cleared to ‘0’, then the I/O Submission Queues and I/O Completion Queues
contained in the Controller Memory Buffer are aligned as defined by the PRP1 field of a
Create I/O Submission Queue command (refer to Figure 477) or a Create I/O Completion
Queue command (refer to Figure 473).
07 RO Impl
Spec
CMB Data Metadata Mixed Memory Support (CDMMMS):  If this bit is set to ‘1’, then
the restriction on data and metadata use of Controller Memory Buffer by a command as
defined in section 8.2.1 is not enforced. If this bit is cleared to ‘0’, then the restriction on
data and metadata use of Controller Memory Buffer by a command as defined in section
8.2.1 is enforced.
06 RO Impl
Spec
CMB Data Pointer and Command Independent Locations Support (CDPCILS): If this
bit is set to ‘1’, then the restriction that the PRP Lists and SGLs shall not be located in
the Controller Memory Buffer if the command that they are associated with is not located
in the Controller Memory Buffer is not enforced (refer to section 8.2.1). If this bit is cleared
to ‘0’, then that restriction is enforced.
05 RO Impl
Spec
CMB Data Pointer Mixed Locations Support (CDPMLS):  If this bit is set to ‘1’, then
the restriction that for a particular PRP List or SGL associated with a single command,
all memory that contains that particular PRP List or SGL shall reside in either the
Controller Memory Buffer or outside the Controller Memory Buffer, is not enforced (refer
to section 8.2.1). If this bit is cleared to ‘0’, then that restriction is enforced.
