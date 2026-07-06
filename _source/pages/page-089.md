---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 89
headings: ["3.1.4.15 Offset 48h: BPMBL – Boot Partition Memory Buffer Location", "3.1.4.16 Offset 50h: CMBMSC – Controller Memory Buffer Memory Space Control"]
---

# Source page 89

NVM Express® Base Specification, Revision 2.1

67
 Offset 48h: BPMBL – Boot Partition Memory Buffer Location
This optional property specifies the memory buffer that is used as the destination for data when a Boot
Partition is read (refer to section 8.1.3). If the controller does not support the Boot Partitions feature, then
this property shall be cleared to 0h.
Figure 51: Offset 48h: BPMBL – Boot Partition Memory Buffer Location
Bits Type Reset Description
63:12 RW 0h Boot Partition Memory Buffer Base Address (BMBBA):  This field specifies the 52
most significant bits of the 64-bit physical address for the Boot Partition Memory Buffer.
11:00 RO 0h Reserved
 Offset 50h: CMBMSC – Controller Memory Buffer Memory Space Control
This optional property specifies how the controller references the Controller Memory Buffer with host -
supplied addresses. If the controller supports the Controller Memory Buffer (CAP.CMBS), this property is
mandatory. Otherwise, this property is reserved.
This property shall be reset by Controller Level Resets other than Controller Lever Resets caused by:
• a Controller Reset; and
• a Function Level Reset (refer to the NVM Express NVMe over PCIe Transport Specification).
Figure 52: Offset 50h: CMBMSC – Controller Memory Buffer Memory Space Control
Bits Type Reset Description
63:12 RW 0h
Controller Base Address (CBA): This field specifies the 52 most significant bits of the
64-bit base address for the Controller Memory Buffer’s controller address range. The
Controller Memory Buffer’s controller base address and its size determine its controller
address range.
The specified address shall be valid only under the following conditions:
a) no part of the Controller Memory Buffer’s controller address range is greater
than 264 − 1; and
b) if the Persistent Memory Region’s controller memory space is enabled, then the
Controller Memory Buffer’s controller address range does not overlap the
Persistent Memory Region’s controller address range.
11:02 RO 0h Reserved
01 RW 0b
Controller Memory Space Enable (CMSE):  This bit specifies whether addresses
supplied by the host are permitted to reference the Controller Memory Buffer.
If CMBSMSC.CRE is cleared to ‘0’ this bit has no effect, and the Controller Memory
Buffer’s controller memory space is not enabled.
If this bit is set to ‘1’ and the controller base address is valid, then the Controller Memory
Buffer’s controller memory space is enabled. Otherwise, the controller memory space is
not enabled.
If the Controller Memory Buffer’s controller memory space is enabled, then addresses
supplied by the host that fall within the Controller Memory Buffer’s controller address
range shall reference the Controller Memory Buffer.
If the Controller Memory Buffer’s controller memory space is not enabled, then no
address supplied by the host shall reference the Controller Memory Buffer. Instead, such
addresses shall reference memory spaces other than the Controller Memory Buffer.
00 RW 0b
Capabilities Registers Enabled (CRE):  This bit specifies whether the CMBLOC and
CMBSZ properties are enabled. If this bit is set to ‘1’, then CMBLOC is defined as shown
in Figure 47 and CMBSZ is defined as shown in Figure 48. If this bit is cleared to ‘0’, then
CMBSZ and CMBLOC are cleared to 0h.
