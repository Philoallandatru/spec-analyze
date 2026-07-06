---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 354
headings: ["5.1.13.2.4 NVM Set List (CNS 04h)"]
---

# Source page 354

NVM Express® Base Specification, Revision 2.1

332
The controller may return any number of variable length Namespace Identification Descriptor structures
that fit into the 4,096 byte Identify payload. All remaining bytes after the Namespace Identification Descriptor
structures should be cleared to 0h, and the host shall interpret a Namespace Identifier Descriptor Length
(NIDL) value of 0h as the end of the list. The host should ignore any Namespace Identification Descriptor
with a Namespace Identifier Type not supported by the host.
A controller shall not return multiple  Namespace Identification  Descriptors with the same Namespace
Identifier Type (NIDT). A controller shall return:
• at least one Namespace Identification Descriptor identifying the namespace (i.e., NIDT field set to
1h, 2h, or 3h); and
• A Namespace Identification Descriptor identifying the I/O Command Set (i.e., NIDT field set to 4h)
if the CAP.CSS.IOCSS bit is set to ‘1’.
Figure 315: Identify – Namespace Identification Descriptor
Bytes Description
00
Namespace Identifier Type (NIDT): This field indicates the data type contained in the Namespace
Identifier field and the length of that type as defined in the following table.
Value Length
(NIDL) Definition
0h  Reserved
1h 8h
IEEE Extended Unique Identifier  (IEUID): The NID field contains a
copy of the EUI64 field in the Identify Namespace data structure (refer
to the applicable I/O Command Set specification). If the EUI64 field of
the Identify Namespace data structure is not supported, (i.e., EUI64
field is cleared to 0h), the controller shall not report a Namespace
Identification Descriptor with a value of type 1h.
2h 10h
Namespace Globally Unique Identifier  (NGUID): The NID field
contains a copy of the NGUID field in the Identify Namespace data
structure (refer to the applicable I/O Command Set specification). If the
NGUID field of the Identify Namespace data structure is not supported
(i.e., the NGUID field is cleared to 0h) , the controller shall not report a
Namespace Identification Descriptor with a value of type 2h.
3h 10h
Namespace UUID  (NUUID): The NID field contains a 128 -bit
Universally Unique Identifier (UUID) as specified in RFC 9562. Refer to
section 4.5.6.
If the namespace does not support an IEEE Extended Unique Identifier
(i.e., EUI64 field is cleared to 0h) and does not support a Namespace
Globally Unique Identifier (i.e., the NGUID field is cleared to 0h), then
the namespace shall report a Namespace Identification Descriptor with
a value of type 3h.
4h 1h
Command Set Identifier (CSI): The NID field contains the Command
Set Identifier that indicates the I/O Command Set that is associated
with this namespace. Refer to Figure 311.
5h to FFh  Reserved

01
Namespace Identifier Length (NIDL): This field contains the length in bytes of the Namespace
Identifier (NID) field. The total length of the Namespace Identification Descriptor in bytes is the
value in this field plus four. If this field is cleared to 0h, then the end of the Namespace Identifier
Descriptor list.
03:02 Reserved
(NIDL + 3):04
Namespace Identifier (NID): For an NIDT field value of 1h, 2h, and 3h, this field contains a value
that is globally unique. The type of the value is specified by the Namespace Identifier Type (NIDT)
field, and the size is specified by the Namespace Identifier Length (NIDL) field.
5.1.13.2.4 NVM Set List (CNS 04h)
Figure 317 defines an NVM Set List. The data structure is an ordered list  of NVM Set Attribute Entry data
structures, sorted by NVM Set Identifier, starting with the first NVM Set Identifier supported by the NVM
subsystem that is equal to or greater than the NVM Set Identifier specified by the CNS Specific Identifier
