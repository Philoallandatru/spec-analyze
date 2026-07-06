---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 178
headings: []
---

# Source page 178

NVM Express® Base Specification, Revision 2.1

156
Figure 118: SGL Bit Bucket descriptor
Bytes Description
11:08
Length (LEN): This field specifies the amount of source data that is discarded. An SGL Bit Bucket
descriptor specifying that no source data be discarded (i.e., this field cleared to 0h) is a valid SGL Bit
Bucket descriptor.
If the SGL Bit Bucket descriptor describes a destination data buffer (e.g., a read from the controller
to host memory), then this field specifies the number of bytes of the source data which the controller
shall discard (i.e., not transfer to the destination data buffer).
If the SGL Bit Bucket descriptor describes a source data buffer (e.g., a write from host memory to the
controller), then the Bit Bucket Descriptor shall be treated as if this field were cleared to 0h (i.e., the
Bit Bucket Descriptor has no effect).
If SGL Bit Bucket descriptors are supported, their length in a destination data buffer shall be included
in the specified length of data to be transferred (e.g., their length in a source data buffer is not included
in the transfer length specified by the NLB parameter).
14:12 Reserved
15
SGL Identifier (SGLID): The definition of this field is described in the table below.
Bits Description
07:04 SGL Descriptor Type (SGLDT): 1h as specified in Figure 115.
03:00 SGL Descriptor Sub Type (SGLDST): Valid values are specified in Figure 116.

The SGL Segment descriptor, defined in Figure 119, describes the next SGL segment, which is not the last
SGL segment.
Figure 119: SGL Segment descriptor
Bytes Description
07:00
Address (ADDR): If the SGL Descriptor Sub Type field is cleared to 0h  in the SGL Identifier field,
then this field specifies the starting 64 -bit memory byte address of the next SGL segment, which is
an SGL segment. If the SGL Descriptor Sub Type field is set to 1h  in the SGL Identifier field, then
this field contains an offset from the beginning of the location where data may be transferred.
11:08
Length (LEN): This field specifies the length in bytes of the next SGL segment. This field shall be a
non-zero value and a multiple of 16.
If the value in the Address field plus the value in this field is greater than 1_00000000_00000000h,
then the SGL Segment descriptor shall be processed as having a Data SGL Length Invalid or
Metadata SGL Length Invalid error.
14:12 Reserved
15
SGL Identifier (SGLID): The definition of this field is described in the table below.
Bits Description
07:04 SGL Descriptor Type (SGLDT): 2h as specified in Figure 115.
03:00 SGL Descriptor Sub Type (SGDST): Valid values are specified in Figure 116.

The SGL Last Segment descriptor, defined in Figure 120, describes the next and last SGL segment. A last
SGL segment that contains an SGL Segment descriptor or an SGL Last Segment descriptor is processed
as an error.
Figure 120: SGL Last Segment descriptor
Bytes Description
07:00
Address (ADDR): If the SGL Identifier Descriptor Sub Type field is cleared to 0h, then this field
specifies the starting 64-bit memory byte address of the next and last SGL segment, which is an SGL
segment. If the SGL Identifier Descriptor Sub Type field is set to 1h, then this field contains an offset
from the beginning of the location where data may be transferred.
