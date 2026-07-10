---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 639
headings: ["8.3.2.3.3 Zone"]
---

# Source page 639

NVM Express® Base Specification, Revision 2.1

617
Figure 682: ZoneGroup Detailed Representation
Bytes Description
255+LenZ1+LenZ2:256+LenZ1
Zone 2 (Z2): This field contains the second Zone contained in
this ZoneGroup as defined in Figure 683 (if present). The length
of this field is represented as LenZ2.
… …
255+LenZ1+…+LenZn:256+LenZ1+…+LenZn-1
Zone N (ZN): This field contains the Nth Zone contained in this
ZoneGroup as defined in Figure 683 (if present). The length of
this field is represented as LenZn.
A ZoneGroup is uniquely identified by the pair {ZoneGroup Name, ZoneGroup Originator}. For each
ZoneGroup, the CDC shall maintain:
• a unique ZoneGroup key, used as a compact ZoneGroup identifier in the FZS and FZR commands
(refer to sections 5.3.6 and 5.3.5 respectively). ZoneGroup keys should not be reused; and
• a generation number, incremented each time a ZoneGroup is updated and used by the GAZ
operation (refer to section 8.3.2.3.8.2). If the value of the generation number is FFFFFFFFh, then
the generation number shall be set to 1h when incremented (i.e., rolls over to 1h).
8.3.2.3.3 Zone
A Zone is the unit of access control. Zone members belonging to the same Zone are allowed to
communicate between each other, according to the rules specified in section 8.3.2.3.4.1. The detailed
representation of a Zone is shown in Figure 683.
Figure 683: Zone Detailed Representation
Bytes Description
 123:00
Zone Name (ZNAME):  This field contains an ASCII encoded null -
terminated string, NULL padded as necessary to the 124-byte maximum
length.
125:124
Number of Zone Members (NUMZM):  This field specifies the number
of Zone members contained in this Zone.  The value of this field is
represented as n.
127:126
Number of Zone Properties (NUMZP): This field specifies the number
of Zone properties contained in this Zone.  The value of this field is
represented as k.
383:128 Zone Member 1 (ZM1):  This field contains the first Zone member
contained in this Zone.
639:384 Zone Member 2 (ZM2):  This field contains the second Zone member
contained in this Zone.
… …
256*n+127:256*n-128 Zone Member n (ZMn): This field contains the Nth Zone member
contained in this Zone (if present).
256*n+127+LenP1:256*n+128
Zone Property 1 (ZP1):  This field contains the first Zone property
contained in this Zone (if present). The length of this field is represented
as LenP1.
256*n+127+LenP1+LenP2:
256*n+128+LenP1
Zone Property 2 (ZP2):  This field contains the second Zone property
contained in this Zone (if present). The length of this field is represented
as LenP2.
… …
256*n+127+LenP1+LenP2+…+LenPk:
256*n+128+LenP1+LenP2+…+LenPk-1
Zone Property k (ZPk): This field contains the Kth Zone property
contained in this Zone (if present). The length of this field is represented
as LenPk.
