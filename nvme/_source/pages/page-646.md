---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 646
headings: ["8.3.2.3.5.2 Fabric Enforced Zone Property", "8.3.2.3.6 ZoneAlias", "8.3.2.3.7 ZoneAlias Members", "8.3.2.3.7.1 Overview", "8.3.2.3.8 Zoning Operations", "8.3.2.3.8.1 Overview"]
---

# Source page 646

NVM Express® Base Specification, Revision 2.1

624
Figure 694: Zone Property Types
Type Definition
FEh to F0h Vendor specific
All others Reserved
 Fabric Enforced Zone Property
This Zone property specifies if a Zone is intended to be enforced through packet -by-packet network level
restrictions. The format of this Zone property is shown in Figure 695.
Figure 695: Fabric Enforced Zone Property Format
Bytes Description
00 Zone Property Type (ZPTYPE): This field specifies the Zone property type and shall be set to 1h.
02:01 Zone Property Length (ZPLEN): This field specifies the length in bytes of this Zone property type and
shall be set to 4h.
03
Zone Property Value (ZPVAL):  This field specifies the value of this Zone property. This field shall be
set to the following value:
• 1h: Network level enforcement requested.
All other values are ignored.
8.3.2.3.6 ZoneAlias
A ZoneAlias is a convenient grouping of NVMe entities identified and is referenceable by a ZoneAlias name.
The detailed representation of a ZoneAlias is shown in Figure 696.
Figure 696: ZoneAlias Detailed Representation
Bytes Description
123:00 ZoneAlias Name (ZANAME):  This field contains an ASCII encoded null -terminated string, NULL
padded as necessary to the 124-byte maximum length.
125:124 Number of ZoneAlias Members (NUMZAM): This field specifies the number of ZoneAlias members
contained in this ZoneAlias. The value of this field is represented as n.
127:126 Reserved
ZoneAlias Member List
383:128 ZoneAlias Member 1: This field contains the first ZoneAlias member contained in this ZoneAlias.
639:384 ZoneAlias Member 2: This field contains the second ZoneAlias member contained in this ZoneAlias
(if present).
… …
256*n+127:
256*n-128
ZoneAlias Member n: This field contains the last ZoneAlias member contained in this ZoneAlias (if
present).
8.3.2.3.7 ZoneAlias Members
 Overview
ZoneAlias members are represented as type-length-value (TLV) data structures. ZoneAlias members may
be any of the Zone member types specified in Figure 684 except the ZoneAlias Zone member type (i.e.,
ZMTYPE field set to EFh).
8.3.2.3.8 Zoning Operations
 Overview
ZoneGroups are data structures maintained and managed (i.e., created, read, modified, or deleted) in the
Zoning database of the CDC. During access to a ZoneGroup (e.g., the ZoneGroup is created or modified)
through an administrative interface (e.g., an adm inistrative interface on the CDC outside the scope of this
