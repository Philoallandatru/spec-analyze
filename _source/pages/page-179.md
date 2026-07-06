---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 179
headings: ["4.3.2.1 SGL Example"]
---

# Source page 179

NVM Express® Base Specification, Revision 2.1

157
Figure 120: SGL Last Segment descriptor
Bytes Description
11:08
Length (LEN): This field specifies the length in bytes of the next and last SGL segment. This field
shall be a non-zero value and a multiple of 16.
If the value in the Address field plus the value in this field is greater than 1_00000000_00000000h,
then the SGL Last Segment descriptor shall be processed as having a Data SGL Length Invalid or
Metadata SGL Length Invalid error.
14:12 Reserved
15
SGL Identifier (SGLID): The definition of this field is described in the table below.
Bits Description
07:04 SGL Descriptor Type (SGLDT): 3h as specified in Figure 115.
03:00 SGL Descriptor Sub Type (SGLDST): Valid values are specified in Figure 116.

The Keyed SGL Data Block descriptor, defined in Figure 121, describes a keyed data block.
Figure 121: Keyed SGL Data Block descriptor
Bytes Description
07:00 Address (ADDR): This field specifies the starting 64-bit memory byte address of the data block.
10:08
Length (LEN): This field specifies the length in bytes of the data block. This field cleared to 0h
specifies that no data is transferred. An SGL Data Block descriptor specifying that no data is
transferred is a valid SGL Data Block descriptor.
If the value in the Address field plus the value in this field is greater than 1_00000000_00000000h,
then the SGL Data Block descriptor shall be processed as having a Data SGL Length Invalid or
Metadata SGL Length Invalid error.
14:11 Key (KEY): Specifies a 32-bit key that is associated with the data block.
15
SGL Identifier (SGLID): The definition of this field is described in the table below.
Bits Description
07:04 SGL Descriptor Type (SGLDT): 4h as specified in Figure 115.
03:00 SGL Descriptor Sub Type (SGLDST): Valid values are specified in Figure 116.


Figure 122: Transport SGL Data Block descriptor
Bytes Description
07:00 Reserved
11:08
Length (LEN): This field specifies the length in bytes of the data block. This field cleared to 0h
specifies that no data is transferred. A Transport SGL Data Block descriptor specifying that no data
is transferred is a valid Transport SGL Data Block descriptor. If the controller requires dword
alignment and granularity as specified in the SGL Support field of Identify Controller (refer to Figure
312), then the two least significant bits shall be cleared to 00b.
The data transfer mechanism and data buffers for data specified by a Transport SGL Data Block
descriptor are defined by the binding section for the associated NVMe Transport.
14:12 Reserved
15
SGL Identifier (SGLID): The definition of this field is described in the table below.
Bits Description
07:04 SGL Descriptor Type (SGLDT): 5h as specified in Figure 115.
03:00 SGL Descriptor Sub Type (SGLDST): Valid values are specified in Figure 116.

 SGL Example
Figure 123 shows an example of a data read request using SGLs. In the example, the logical block size is
512B. The total length of the logical blocks accessed is 13  KiB, of which only 11  KiB is transferred to the
host. The Number of Logical Blocks (NLB) field in the command shall specify 26, indicating the total length
