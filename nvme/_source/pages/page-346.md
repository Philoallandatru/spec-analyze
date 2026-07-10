---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 346
headings: []
---

# Source page 346

NVM Express® Base Specification, Revision 2.1

324
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
539:536 O O M
SGL Support (SGLS): This field indicates if SGLs are supported and the particular SGL
types supported. Refer to section 4.3.2.
Bits Description
31:22 Reserved
21
Transport SGL Data Block Descriptor Support (TSDBDS): If this bit
is set to ‘1’, then the controller supports the Transport SGL Data Block
descriptor. If this bit is cleared to ‘0’, then the controller does not support
the Transport SGL Data Block descriptor.
20
SGL Address Offset Supported (SAOS):  If this bit is set to ‘1’, then
the controller supports the Address field in SGL Data Block, SGL
Segment, and SGL Last Segment descriptor types specifying an offset.
If this bit is cleared to ‘0’, then the Address field specifying an offset is
not supported.
19
MPTR SGL Descriptor Support (MSDS):  If this bit is set to ‘1’, then
use of a Metadata Pointer (MPTR) that contains an address of an SGL
segment containing exactly one SGL Descriptor that is qword aligned is
supported. If this bit is cleared to ‘0’, then use of a MPTR containing an
SGL Descriptor is not supported.
18
Length Larger than Data Transfer Support (LLDTS): If this bit is set
to ‘1’, then the controller supports commands that contain a data or
metadata SGL of a length larger than the amount of data to be
transferred. If this bit is cleared to ‘0’, then the SGL length shall be equal
to the amount of data to be transferred.
17
Metadata Buffer Alignment (MBA): This bit indicates metadata buffer
alignment requirements when CDW0.PSDT is set to 01b (refer to Figure
91). If set to ‘1’, then use of a byte aligned contiguous physical buffer of
metadata (the Metadata Pointer field in Figure 92) is supported. If
cleared to ‘0’, then use of a byte aligned contiguous physical buffer of
metadata is not supported.
16
SGL Bit Bucket Descriptor Supported (SBBDS):  If this bit is set to
‘1’, then the SGL Bit Bucket descriptor is supported. If this bit is cleared
to ‘0’, then the SGL Bit Bucket descriptor is not supported.
15:08
SGL Descriptor Threshold (SDT):  This field indicates the
recommended maximum number of SGL descriptors in a command
(refer to section 4.3.2). If this field is cleared to 0h, then no
recommended maximum number of SGL descriptors is reported.
07:03 Reserved
02
Keyed SGL Data Block Descriptor Support (KSDBDS):  If this bit is
set to ‘1’, then the controller supports the Keyed SGL Data Block
descriptor. If this bit is cleared to ‘0’, then the controller does not support
the Keyed SGL Data Block descriptor.
01:00
SGL Support (SGLS): This field is used to determine the SGL support.
Valid values are shown in the table below.
Value Definition
00b SGLs are not supported.
01b SGLs are supported. There is no alignment nor
granularity requirement for Data Blocks.
10b
SGLs are supported. There is a dword alignment and
granularity requirement for Data Blocks (refer to section
4.3.2).
11b Reserved
