---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 184
headings: ["4.5.2 Serial Number (SN) and Model Number (MN)", "4.5.3 IEEE OUI Identifier (IEEE)", "4.5.4 IEEE Extended Unique Identifier (EUI64)"]
---

# Source page 184

NVM Express® Base Specification, Revision 2.1

162
Figure 126: PCI Vendor ID (VID) and PCI Subsystem Vendor ID (SSVID)
Bytes 00 01 02 03
Value CDh ABh 34h 12h
 Serial Number (SN) and Model Number (MN)
The Serial Number (SN, bytes 23:04) and Model Number (MN, bytes 63:24) are defined in the Identify
Controller data structure. The values are ASCII strings assigned by the vendor. Each identifier is in big
endian format.
Example (Value shown as ASCII characters):
• SN = “SN1”; and
• MN = “M2”.
Figure 127: Serial Number (SN) and Model Number (MN)
Bytes 04 05 06 23 to 07 24 25 63 to 26
Value 53h (‘S’) 4Eh (‘N’) 31h (‘1’) 20h (‘ ‘) 4Dh (‘M’) 32h (‘2’) 20h (‘ ‘)
 IEEE OUI Identifier (IEEE)
The IEEE OUI Identifier (OUI, bytes 75:73) is defined in the Identify Controller data structure. The value is
assigned by the IEEE Registration Authority. The identifier is in little endian format.
Example:
• OUI = ABCDEFh.
Figure 128: IEEE OUI Identifier (IEEE)
Bytes 73 74 75
Value EFh CDh ABh
 IEEE Extended Unique Identifier (EUI64)
The IEEE Extended Unique Identifier (EUI64, bytes 127:120) is defined in the Identify Namespace data
structure. Tutorials are available at https://standards.ieee.org/develop/regauth/tut/index.html. IEEE defines
three formats that may be used in this field: MA-L, MA-M, and MA-S. The examples in this section use the
MA-L format.
The MA-L format is defined as a string of eight octets:
Figure 129: IEEE Extended Unique Identifier (EUI64), MA-L Format
EUI[0] EUI[1] EUI[2] EUI[3] EUI[4] EUI[5] EUI[6] EUI[7]
OUI Extension Identifier
EUI64 is defined in big endian format. The OUI field differs from the OUI Identifier which is in little endian
format as described in section 4.5.3.
Example:
• OUI Identifier = ABCDEFh; and
• Extension Identifier = 0123456789h.
