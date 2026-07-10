---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 87
headings: ["3.1.4.12 Offset 3Ch: CMBSZ – Controller Memory Buffer Size"]
---

# Source page 87

NVM Express® Base Specification, Revision 2.1

65
Figure 47: Offset 38h: CMBLOC – Controller Memory Buffer Location
Bits Type Reset Description
04 RO Impl
Spec
CMB Queue Physically Discontiguous Support (CQPDS): If this bit is set to ‘1’, then
the restriction that for all queues in the Controller Memory Buffer, the queue shall be
physically contiguous, is not enforced (refer to section 8.2.1). If this bit is cleared to ‘0’,
then that restriction is enforced.
03 RO Impl
Spec
CMB Queue Mixed Memory Support (CQMMS):  If this bit is set to ‘1’, then for a
particular queue placed in the Controller Memory Buffer, the restriction that all memory
associated with that queue shall reside in the Controller Memory Buffer is not enforced
(refer to section 8.2.1). If this bit is cleared to ‘0’, then that requirement is enforced.
02:00 RO Impl
Spec
Base Indicator Register (BIR):  Indicates the Base Address Register (BAR) that
contains the Controller Memory Buffer. For a 64-bit BAR, the BAR for the least significant
32-bits of the address is specified. Values 000b, 010b, 011b, 100b, and 101b are valid.
The address specified by the BAR shall be 4 KiB aligned.
 Offset 3Ch: CMBSZ – Controller Memory Buffer Size
This optional property defines the size of the Controller Memory Buffer (refer to section 8.2.1). If the
controller does not support the Controller Memory Buffer feature or if the controller supports the Controller
Memory Buffer (CAP.CMBS) and CMBMSC.CRE is cleared to ‘0’, then this property shall be cleared to 0h.
Figure 48: Offset 3Ch: CMBSZ – Controller Memory Buffer Size
Bits Type Reset Description
31:12 RO Impl
Spec
Size (SZ): Indicates the size of the Controller Memory Buffer available for use by the
host. The size is in multiples of the Size Unit. If the Offset + Size exceeds the length of
the indicated BAR, the size available to the host is limited by the length of the BAR.
11:08 RO Impl
Spec
Size Units (SZU): Indicates the granularity of the Size field.
Value Granularity
0h 4 KiB
1h 64 KiB
2h 1 MiB
3h 16 MiB
4h 256 MiB
5h 4 GiB
6h 64 GiB
7h to Fh Reserved

07:05 RO 000b Reserved
04 RO Impl
Spec
Write Data Support (WDS): If this bit is set to ‘1’, then the controller supports data and
metadata in the Controller Memory Buffer for commands that transfer data from the host
to the controller (e.g., Write). If this bit is cleared to ‘0’, then data and metadata for
commands that transfer data from the host to the controller shall not be transferred to
the Controller Memory Buffer.
03 RO Impl
Spec
Read Data Support (RDS): If this bit is set to ‘1’, then the controller supports data and
metadata in the Controller Memory Buffer for commands that transfer data from the
controller to the host (e.g., Read). If this bit is cleared to ‘0’, then data and metadata for
commands that transfer data from the controller to the host shall not be transferred from
the Controller Memory Buffer.
02 RO Impl
Spec
PRP SGL List Support (LISTS): If this bit is set to ‘1’, then the controller supports PRP
Lists in the Controller Memory Buffer. If this bit is set to ‘1’ and SGLs are supported by
the controller, then the controller supports Scatter Gather Lists in the Controller Memory
Buffer. If this bit is set to ‘1’, then the Submission Queue Support bit shall be set to ‘1’. If
this bit is cleared to ‘0’, then PRP Lists and SGLs shall not be placed in the Controller
Memory Buffer.
01 RO Impl
Spec
Completion Queue Support (CQS): If this bit is set to ‘1’, then the controller supports
Admin and I/O Completion Queues in the Controller Memory Buffer. If this bit is cleared
to ‘0’, then Completion Queues shall not be placed in the Controller Memory Buffer.
