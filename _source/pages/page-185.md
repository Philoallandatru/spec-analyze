---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 185
headings: ["4.5.5 Namespace Globally Unique Identifier (NGUID)"]
---

# Source page 185

NVM Express® Base Specification, Revision 2.1

163
Figure 130: IEEE Extended Unique Identifier (EUI64), OUI Identifier
Bytes 120 121 122 123 124 125
Value ABh CDh EFh 01h 23h 45h
Field OUI Extension Identifier

Figure 131: IEEE Extended Unique Identifier (EUI64), Ext. ID (cont)
Bytes 126 127
Value 67h 89h
Field Ext ID (cont)
The MA-L format is similar to the World Wide Name (WWN) format defined as IEEE Registered designator
(NAA = 5) as shown below.
Figure 132: MA-L similarity to WWN
Bytes 0 1 2 3 4 5 6 7
EUI64 OUI Extension Identifier
WWN
(NAA = 5) 5h OUI Vendor Specific Identifier
 Namespace Globally Unique Identifier (NGUID)
The Namespace Globally Unique Identifier (NGUID, bytes 119:104) is defined in the Identify Namespace
data structure. The NGUID is composed of an IEEE OUI, an extension identifier, and a vendor specific
extension identifier. The extension identifier and ven dor specific extension identifier are both assigned by
the vendor and may be considered as a single field. NGUID is defined in big endian format. The OUI field
differs from the OUI Identifier which is in little endian format as described in section 4.5.3.
Example:
• OUI Identifier = ABCDEFh;
• Extension Identifier = 0123456789h; and
• Vendor Specific Extension Identifier = FEDCBA9876543210h.
Figure 133: Namespace Globally Unique Identifier (NGUID)
Bytes 104 105 106 107 108 109
Value FEh DCh BAh 98h 76h 54h
Field Vendor Specific Extension Identifier

Figure 134: Namespace Globally Unique Identifier (NGUID), OUI
Bytes 110 111 112 113 114 115
Value 32h 10h ABh CDh EFh 01h
Field VSP Ex ID (cont) OUI Ex ID

Figure 135: Namespace Globally Unique Identifier
(NGUID), Extension Identifier (continued)
Bytes 116 117 118 119
Value 23h 45h 67h 89h
Field Extension Identifier (continued)
The NGUID format is similar to the World Wide Name (WWN) format as IEEE Registered Extended
designator (NAA = 6) as shown below.
