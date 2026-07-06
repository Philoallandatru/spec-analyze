---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 176
headings: []
---

# Source page 176

NVM Express® Base Specification, Revision 2.1

154
Figure 113: SGL Segment
Bytes Description
SGL Descriptor List
15:00 SGL Descriptor 0: This field is the first SGL descriptor.
31:16 SGL Descriptor 1: This field is the second SGL descriptor.
… …
((n*16)+15):(n*16) SGL Descriptor n: This field is the last SGL descriptor.
An SGL segment contains one or more SGL descriptors. Figure 114 defines the generic SGL descriptor
format.
Figure 114: Generic SGL Descriptor Format
Bytes Description
14:00 Descriptor Type Specific (DTS): This field is descriptor type specific.
15
SGL Identifier (SGLID): The definition of this field is described in the table below.
Bits Description
07:04 SGL Descriptor Type (SGLDT): Refer to Figure 115.
03:00 SGL Descriptor Sub Type (SGLDST): Refer to Figure 116.

The SGL Descriptor Type field defined in Figure 115 specifies the SGL descriptor type. If the SGL Descriptor
Type field is set to a reserved value or an unsupported value, then the SGL descriptor shall be processed
as having an SGL Descriptor Type error. If the SGL Descriptor Sub Type field is set to a res erved value or
an unsupported value, then the descriptor shall be processed as having an SGL Descriptor Type error.
An SGL descriptor set to all zeroes is an SGL Data Block descriptor with the Address field cleared to 0h
and the Length field cleared to 0h may be used as a NULL descriptor.
Figure 115: SGL Descriptor Type
Value Descriptor
0h SGL Data Block descriptor
1h SGL Bit Bucket descriptor
2h SGL Segment descriptor
3h SGL Last Segment descriptor
4h Keyed SGL Data Block descriptor
5h Transport SGL Data Block descriptor
6h to Eh Reserved
Fh Vendor specific
Figure 116 defines the SGL Descriptor Sub Type values and indicates the SGL Descriptor Types to which
each SGL Descriptor Sub Type applies.
Figure 116: SGL Descriptor Sub Type Values
SGL Descriptor
Sub Type
SGL Descriptor
Types  Sub Type Description
0h
0h, 2h, 3h, 4h Address: The Address field specifies the starting 64-bit memory byte address
of the Data Block, Segment, or Last Segment descriptor.
1h For Type 1h, the Sub Type field shall be cleared to 0h.
All other values Reserved
1h 0h, 2h, 3h
Offset: The Address field contains an offset from the beginning of the location
where data may be transferred. For NVMe over PCIe implementations, this
Sub Type is reserved. For NVMe over Fabrics implementations, refer to
section 3.3.2.1.3.1.
