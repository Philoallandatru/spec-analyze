---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 88
headings: ["3.1.4.13 Offset 40h: BPINFO – Boot Partition Information", "3.1.4.14 Offset 44h: BPRSEL – Boot Partition Read Select"]
---

# Source page 88

NVM Express® Base Specification, Revision 2.1

66
Figure 48: Offset 3Ch: CMBSZ – Controller Memory Buffer Size
Bits Type Reset Description
00 RO Impl
Spec
Submission Queue Support (SQS): If this bit is set to ‘1’, then the controller supports
Admin and I/O Submission Queues in the Controller Memory Buffer. If this bit is cleared
to ‘0’, then Submission Queues shall not be placed in the Controller Memory Buffer.
 Offset 40h: BPINFO – Boot Partition Information
This optional property defines the characteristics of Boot Partitions (refer to section 8.1.3). If the controller
does not support the Boot Partitions feature, then this property shall be cleared to 0h.
Figure 49: Offset 40h: BPINFO – Boot Partition Information
Bits Type Reset Description
31 RO Impl
Spec
Active Boot Partition ID (ABPID): This bit indicates the identifier of the active Boot
Partition.
30:26 RO 0h Reserved
25:24 RO 00b
Boot Read Status (BRS): This field indicates the status of Boot Partition read
operations initiated by the host writing to the BPRSEL.BPID field. Refer to section
8.1.3.
The boot read status values are defined as:
Value Definition
00b No Boot Partition read operation requested
01b Boot Partition read in progress
10b Boot Partition read completed successfully
11b Error completing Boot Partition read
If host software writes the BPRSEL.BPID field, this field transitions to 01b. After
successfully completing a Boot Partition read operation (i.e., transferring the contents
to the boot memory buffer), the controller sets this field to 10b. If there is an error
completing a Boot Partition read operation, this field is set to 11b, and the contents
of the boot memory buffer are undefined.
23:15 RO 0h Reserved
14:00 RO Impl
Spec
Boot Partition Size (BPSZ): This field defines the size of each Boot Partition in
multiples of 128 KiB. Both Boot Partitions are the same size.
 Offset 44h: BPRSEL – Boot Partition Read Select
This optional property is used to initiate the transfer of a data in the Boot Partition (refer to section 8.1.3)
from the controller to the host. If the controller does not support the Boot Partitions feature, then this property
shall be cleared to 0h.
If the host attempts to read beyond the end of a Boot Partition (i.e., the Boot Partition Read Offset plus Boot
Partition Read Size, is greater than the Boot Partition Size in bytes), the controller shall not transfer data
and report an error in the BPINFO.BRS field.
Figure 50: Offset 44h: BPRSEL – Boot Partition Read Select
Bits Type Reset Description
31 RW 0b Boot Partition Identifier (BPID): This bit specifies the Boot Partition identifier for the
Boot Partition read operation.
30 RO 0b Reserved
29:10 RW 0h
Boot Partition Read Offset (BPROF):  This field selects the offset into the Boot
Partition, in 4  KiB units, that the controller copies into the Boot Partition Memory
Buffer.
09:00 RW 0h Boot Partition Read Size (BPRSZ):  This field selects the read size in multiples of
4 KiB to copy into the Boot Partition Memory Buffer.
