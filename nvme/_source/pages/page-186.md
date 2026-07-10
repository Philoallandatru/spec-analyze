---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 186
headings: ["4.5.6 Universally Unique Identifier (UUID)", "4.6 List Data Structures", "4.6.1 Controller List", "4.6.2 Namespace List"]
---

# Source page 186

NVM Express® Base Specification, Revision 2.1

164
Figure 136: Namespace Globally Unique Identifier (NGUID), NGUID similarity to WWN
Bytes 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
NGUID Vendor Specific Extension Identifier OUI Extension Identifier
WWN
(NAA = 6) 6h OUI Vendor Specific Identifier Vendor Specific Identifier Extension
 Universally Unique Identifier (UUID)
The Universally Unique Identifier is defined in RFC 9562 and contained in the Namespace Identification
Descriptor (refer to Figure 315). Byte ordering requirements for a UUID are described in RFC 9562.
For historical reasons, UUID subtypes are called UUID versions. Multiple UUID versions are able to be
used in the same implementation.
RFC 9562 defines UUID version 8 (i.e., UUIDv8) for experimental or vendor-specific use cases. Uniqueness
of UUIDv8 values is implementation specific. NVM Express UUID use cases assume uniqueness within the
set of hosts, NVM subsystems, and fabrics that are connected or accessible by a common instance of an
administrative tool. For UUIDv8 values, that uniqueness is the responsibi lity of all of the implementations
in that set. RFC 9562 Appendix B provides examples of UUIDv8 generation algorithms that produce unique
UUIDs if all implementations in that set generate UUIDv8 values with the same algorithm.
4.6 List Data Structures
This section describes list data structures used in this specification.
 Controller List
A Controller List, defined in Figure 137, is an ordered list of ascending controller identifiers. The controller
identifier  of a controller is the value indicated in the CNTLID field of the Identify Controller data structure in
Figure 312. Unused entries are zero filled.
Figure 137: Controller List Format
Bytes Description
Header
01:00
Number of Controller Identifiers (NUMCIDS): This field contains the number of controller
entries in the list. There may be up to 2,047 identifiers in the list. A value of 0h indicates there
are no controllers in the list.
Controller Identifier List
03:02 Controller Identifier 0: This field contains the NVM subsystem unique controller identifier for
the first controller in the list, if present.
05:04 Controller Identifier 1: This field contains the NVM subsystem unique controller identifier for
the second controller in the list, if present.
… …
(NUMCIDS*2+1):
(NUMCIDS*2)
Controller Identifier NUMCIDS-1: This field contains the NVM subsystem unique controller
identifier for the last controller in the list, if present.
 Namespace List
A Namespace List, defined in Figure 138, is an ordered list of namespace IDs. Unused entries are zero
filled.
Figure 138: Namespace List Format
Bytes Description
Namespace Identifier List
03:00 Namespace Identifier 0: This field contains the lowest namespace ID in the list or 0h if the list is
empty.
