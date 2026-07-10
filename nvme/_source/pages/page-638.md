---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 638
headings: ["8.3.2.3.2 ZoneGroup"]
---

# Source page 638

NVM Express® Base Specification, Revision 2.1

616
Figure 680: ZoneDBActive Abstract Representation
ZoneDBActive
ZoneGroup#1
ZoneGroup#2
ZoneGroup#n

The ZoneDBConfig is a list of configured ZoneGroups and ZoneAliases, as shown in Figure 681.
Figure 681: ZoneDBConfig Abstract Representation
ZoneDBConfig
ZoneAlias#1
ZoneAlias#2
ZoneAlias#p

ZoneGroup#1
ZoneGroup#2
ZoneGroup#m


8.3.2.3.2 ZoneGroup
A ZoneGroup is the unit of activation (i.e., a set of access control rules enforceable by the CDC). The
detailed representation of a ZoneGroup is shown in Figure 682.
Figure 682: ZoneGroup Detailed Representation
Bytes Description
223:00
ZoneGroup Originator (ZGORIG): This field contains the NQN
(represented as a null -terminated string, NULL padded as
necessary to the 224-byte maximum length) of either:
• the CDC, if this ZoneGroup was created on the CDC;
or
• the DDC that created this ZoneGroup, if this
ZoneGroup was created by a DDC.
253:224
ZoneGroup Name (ZGNAME):  This field contains an ASCII
encoded null-terminated string, NULL padded as necessary to
the 30-byte maximum length.
255:254
Number of Zones (NUMZ):  This field specifies the number of
Zones contained in this ZoneGroup.  The value of this field is
represented as n.
255+LenZ1:256
Zone 1 (Z1): This field contains the first Zone contained in this
ZoneGroup as defined in Figure 683. The length of this field is
represented as LenZ1.
