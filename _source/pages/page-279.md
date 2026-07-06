---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 279
headings: []
---

# Source page 279

NVM Express® Base Specification, Revision 2.1

257
Figure 257: Capacity Configuration Descriptor
Bytes Description
Header
1:0 Capacity Configuration Identifier  (CCID): This field indicates the identifier for this Capacity
Configuration.
3:2
Domain Identifier (DID): This field indicates the identifier of the domain (refer to section 3.2.5.3)
containing the Endurance Group configurations described by this Capacity Configuration
Descriptor. A value of 0h indicates that a Domain Identifier is not reported. If multiple domains are
not supported, then this field shall be cleared to 0h.
5:4
Number of Endurance Group Configuration Descriptors (EGCN):  This field indicates the
number of Endurance Group Configuration Descriptors in the list. If this field is cleared to 0h, then
no Endurance Group Configuration Descriptors are reported.
31:6 Reserved
Endurance Group Configuration Descriptor List
NOTE 1 Endurance Group Configuration 0 Descriptor:  This field indicates the first Endurance Group
Configuration Descriptor (refer to Figure 258) in the list, if any.
NOTE 1 Endurance Group Configuration 1 Descriptor: This field indicates the second Endurance Group
Configuration Descriptor in the list, if any.
…
NOTE 1 Endurance Group Configuration EGCN -1 Descriptor: This field indicates the last Endurance
Group Configuration Descriptor in the list, if any.
Notes:
1. Endurance Group Configuration Descriptors may be different lengths.
The Endurance Group Configuration Descriptor is defined in Figure 258.
In the Endurance Group Configuration Descriptor  (refer to Figure 258), NVM Set Identifiers shall be listed
in ascending order by value, and each NVM Set Identifier shall appear only once.
In the Endurance Group Configuration Descriptor, Channel Configuration Descriptors shall be listed in
ascending order by Channel Identifier value, and each Channel Identifier shall appear only once.
Figure 258: Endurance Group Configuration Descriptor
Bytes Description
Header
1:0
Endurance Group Identifier (ENDGID): This field indicates the identifier of the Endurance
Group (refer to section 3.2.3) described by this Endurance Group Configuration Descriptor.
This field shall indicate a value greater than or equal to 1h and less than or equal to the value
of the Endurance Group Identifier Maximum field (refer to Figure 312).
3:2
Capacity Adjustment Factor  (CADJF): This field indicates the capacity adjustment factor
(refer to section 8.1.4.1) for this Endurance Group.
A value of FFFFh indicates that value and all higher values.
A value of 0h indicates that the Capacity Adjustment Factor is not reported.
15:4 Reserved
31:16
Total Endurance Group Capacity (TEGCAP): This field indicates the total NVM capacity in
this Endurance Group. The value is in bytes. If this field is cleared to 0h, then the NVM
subsystem does not report the total NVM capacity in this Endurance Group.
47:32
Spare Endurance Group Capacity (SEGCAP):  This field indicates the spare NVM capacity
in this Endurance Group. The value is in bytes. If this field is cleared to 0h, then the NVM
subsystem does not report the unallocated NVM capacity in this Endurance Group.
