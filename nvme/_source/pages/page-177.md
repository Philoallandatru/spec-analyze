---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 177
headings: []
---

# Source page 177

NVM Express® Base Specification, Revision 2.1

155
Figure 116: SGL Descriptor Sub Type Values
SGL Descriptor
Sub Type
SGL Descriptor
Types  Sub Type Description
1h The controller shall abort the command with the status code of SGL Descriptor
Type Invalid.
4h The controller shall abort the command with the status code of SGL Descriptor
Type Invalid.
All other values Reserved
Ah to Fh All NVMe Transport Specific: The definitions for this range of Sub Types are
defined by the binding section for the associated NVMe Transport.
All other values All Reserved
The SGL Data Block descriptor, defined in Figure 117, describes a data block.
Figure 117: SGL Data Block descriptor
Bytes Description
07:00
Address (ADDR): If the SGL Identifier Descriptor Sub Type field is cleared to 0h, then this field
specifies the starting 64 -bit memory byte address of the data block. If the SGL Identifier Descriptor
Sub Type field is set to 1h, then this field contains an offset from the beginning of the location where
data may be transferred. If the controller requires dword alignment and granularity as indicated in the
SGL Support (SGLS) field of the Identify Controller data structure (refer to Figure 312), then the two
least significant bits shall be cleared to 00b.
If dword alignment and granularity is required, the controller may report an error of Invalid Field in
Command if bits 1:0 are not cleared to 00b. If the controller does not report an error of Invalid Field
in Command, then the controller shall operate as if bits 1:0 are cleared to 00b.
11:08
Length (LEN): This field specifies the length in bytes of the data block. This field cleared to 0h
specifies that no data is transferred. An SGL Data Block descriptor specifying that no data is
transferred is a valid SGL Data Block descriptor. If the controller requires dword alignment and
granularity as specified in the SGL Support (SGLS) field the of Identify Controller data structure, then
the two least significant bits shall be cleared to 00b.
If dword alignment and granularity is required, the controller may report an error of Invalid Field in
Command if bits 1:0 are not cleared to 00b. If the controller does not report an error of Invalid Field
in Command, then the controller shall operate as if bits 1:0 are cleared to 00b.
If the value in the Address field plus the value in this field is greater than 1_00000000_00000000h,
then the SGL Data Block descriptor shall be processed as having a Data SGL Length Invalid or
Metadata SGL Length Invalid error.
14:12 Reserved
15
SGL Identifier (SGLID): The definition of this field is described in the table below.
Bits Description
07:04 SGL Descriptor Type (SGLDT): 0h as specified in Figure 115.
03:00 SGL Descriptor Sub Type (SGLDST): Valid values are specified in Figure 116.

The SGL Bit Bucket descriptor, defined in Figure 118, is used to ignore parts of source data.
Figure 118: SGL Bit Bucket descriptor
Bytes Description
07:00 Reserved
