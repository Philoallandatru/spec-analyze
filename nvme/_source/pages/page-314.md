---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 314
headings: []
---

# Source page 314

NVM Express® Base Specification, Revision 2.1

292
Figure 299: Host Extended Discovery Log Page Entry
Bytes Description
Header
00 Transport Type (TRTYPE):  This field indicates the transport type as defined in the
Transport Type (TRTYPE) field in Figure 294.
01 Address Family (ADRFAM) : This field indicates the address family as defined in the
Address Family (ADRFAM) field in Figure 294.
09:02 Reserved
11:10
Entry Flags (EFLAGS): This field indicates additional information related to the current
entry.
Bits Description
15:3 Reserved
2
No CDC Connectivity (NCC): If this bit is set to ‘1’, then the host that describes
this entry is not currently connected to the CDC. If this bit is cleared to ‘0’, then
the host that describes this entry is currently connected to the CDC. If the
Discovery controller returning this log page is not a CDC, then this bit shall be
cleared to ‘0’ and should be ignored by the host.
1:0 Reserved

255:12 Reserved
511:256
Host NVMe Qualified Name (HOSTNQN):  This field indicates the NVMe Qualified Name
(NQN) that uniquely identifies the host. Refer to section 4.7 for the formatting requirements
for NQNs.
767:512
Transport Address (TRADDR):  This field indicates the address of the host that may be
used for a Connect command as an ASCII string. The Address Family field describes the
reference for parsing this field. Refer to section 1.4.2 for ASCII string requirements. For the
definition of this field, refer to the appropriate NVMe Transport binding specification.
1023:768
Transport Specific Address Subtype (TSAS): This field indicates NVMe Transport specific
information about the address as defined in the Transport Specific Address Subtype (TSAS)
field in Figure 294.
1027:1024 Total Entry Length (TEL): This field indicates the length in bytes of the entire Host Extended
Discovery Log Page Entry.
1029:1028
Number of Extended Attributes (NUMEXAT): This field indicates the number of extended
attributes contained in the Host Extended Discovery Log Page Entry. This field shall be non-
zero (i.e., the Host Extended Discovery Log Page Entry contains at least one extended
attribute containing a Host Identifier as described in this section).
1031:1030 Reserved
Extended Attributes List
((EXATLEN - 1) + 4)
+ 1032:1032
Extended Attribute 0:  This field contains the first extended attribute as defined in Figure
499, where EXATLEN is the size specified in the Extended Attribute Length (EXATLEN) field
of the extended attribute.
… …
TEL - 1:TEL -
(EXATLEN + 4)
Extended Attribute N:  This field contains the Nth extended attribute as defined in Figure
499 (if present), where EXATLEN is the size specified in the Extended Attribute Length
(EXATLEN) field of the extended attribute and TEL is the size specified in the Total Entry
Length (TEL) field.

Figure 300: Host Discovery Log Page
Bytes Description
Header
07:00
Generation Counter (GENCTR):  This field indicates the version of the discovery
information, starting at a value of 0h. For each change in the Host Discovery log page, this
counter is incremented by one. If the value of this field is FFFFFFFF_FFFFFFFFh, then the
field shall be cleared to 0h when incremented (i.e., rolls over to 0h).
15:08 Number of Records (NUMREC):  Indicates the number of records contained in the log
page.
