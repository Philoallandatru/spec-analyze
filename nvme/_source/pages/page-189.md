---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 189
headings: ["4.8 UTF-8 String Processing"]
---

# Source page 189

NVM Express® Base Specification, Revision 2.1

167
UUID. EUI64 is an 8-byte EUI-64 identifier (refer to section 4.5.4), NGUID is a 16-byte identifier based on
EUI-64 (refer to section 4.5.5), and Namespace UUID is a 16 -byte identifier described in RFC 9562 (refer
to section 4.5.6).
When creating a namespace, the controller shall indicate a globally unique namespace identifier in one or
more of the following:
a) the EUI64 field;
b) the NGUID field; or
c) a Namespace Identification Descriptor with the Namespace Identifier Type field set to 3h  (i.e., a
UUID).
Refer to section 8.1.9.2 for additional globally unique namespace identifier requirements related to
dispersed namespaces.
If the EUI64 field is cleared to 0h and the NGUID field is cleared to 0h, then the namespace shall support
a valid Namespace UUID in the Namespace Identification Descriptor data structure.
If the UIDREUSE bit in the NSFEAT field is cleared to ‘0’, then a controller may reuse a non-zero
NGUID/EUI64 value for a new namespace after the original namespace using the value has been deleted.
If the UIDREUSE bit is set to ‘1’, then a controller shall not reuse a non -zero NGUID/EUI64 for a new
namespace after the original namespace using the value has been deleted.
4.8 UTF-8 String Processing
NVMe hosts, controllers and NVM subsystems process (e.g., store, compare) UTF-8 strings used by NVMe
as binary strings without any text processing or text comparison logic that is specific to the Unicode
character set or locale (e.g., check for byte values not used by UTF -8, Unicode normalization, etc.). Any
such text processing:
a) may occur as part of entry of UTF -8 strings into NVMe hosts and NVM subsystems as shown at
points 1 and 3 in Figure 140; and
b) should not occur as part of receiving UTF-8 strings via NVMe, as shown at points 2 and 4 in Figure
140.
Upon entry into the NVMe host (e.g., via a configuration interface at point 1 in Figure 140, described as
“input” in RFC  9562), NVMe host software may process a UTF -8 string (e.g., perform Unicode
normalization). Upon entry into the NVM subsystem (e.g., via a configuration interface at point 3 in Figure
140, described as “input” in RFC  9562), a controller may process a UTF -8 string (e.g., perform Unicode
normalization). Upon receipt by the host (e.g., at point 2 in Figure 140) of a UTF-8 string from the controller,
text processing (e.g., Unicode normalization) should not occur. Upon receipt by the controller (e.g., at point
4 in Figure 140) of a UTF-8 string from the host, text processing (e.g., Unicode normalization) should not
occur.
