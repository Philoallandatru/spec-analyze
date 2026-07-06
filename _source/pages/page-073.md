---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 73
headings: ["3.1.4.1 Offset 0h: CAP – Controller Capabilities"]
---

# Source page 73

NVM Express® Base Specification, Revision 2.1

51
Figure 34: Memory-Based Property Definition
Offset
(OFST)
Size (in
bytes)
I/O
Controller1
Admin.
Controller1
Discovery
Controller1 Name
1000h Variable2 T T T
Transport Specific (e.g., PCIe doorbell
registers as specified in the NVMe over
PCIe Transport Specification)
1000h +
Variable2  O O O Vendor Specific
Notes:
1. O/T definition: O = Optional, T = Transport Specific
2. Determined by the transport (e.g., the offset calculation formula Offset (1000h + ((2y) * (4 << CAP.DSTRD)))
for the PCIe transport).

Figure 35: Message-Based Property Definition
Offset
(OFST)
Size (in
bytes)
I/O
Controller1
Admin.
Controller1
Discovery
Controller1 Name
1000h 300h R R R Reserved for Fabrics
1300h  O O O Vendor Specific
Notes:
1. O/R definition: O = Optional, R = Reserved
The following conventions are used to describe controller properties  for all transport models. Hardware
shall return ‘0’ for all bits that are marked as reserved, and host software shall write all reserved bits and
properties with the value of 0h.
The following terms and abbreviations are used:
RO Read Only
RW Read Write
RWC Read/Write ‘1’ to clear
RWS Read/Write ‘1’ to set
Impl Spec Implementation Specific – the controller has the freedom to choose
its implementation.
HwInit The default state is dependent on NVM Express controller and
system configuration.
Reset This column indicates the value of the field after a Controller Level
Reset as defined in section 3.7.2.
For some fields, it is implementation specific as to whether the field is RW, RWC, or RO; this is typically
shown as RW/RO or RWC/RO to indicate that if the functionality is not supported that the field is read only.
When a field is referred to in the document, the convention used is “ Property Symbol.Field Symbol”. For
example, the PCI command register Parity Error Response Enable bit is referred to by the name CMD.PEE.
If the field is an array of bits, the field is referred to as “ Property Symbol.Field Symbol (array offset to
element)”. When a sub-field is referred to in the document, the convention used is “Property Symbol.Field
Symbol.Sub Field Symbol”. For example, when the Controller Ready With Media Support sub -field of the
Controller Ready Modes Supported field within the Controlle r Capability property, the sub -field is referred
to by the name CAP.CRMS.CRWMS.
 Offset 0h: CAP – Controller Capabilities
This property indicates basic capabilities of the controller to host software.
