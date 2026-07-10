---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 96
headings: ["3.1.4.27 Offset E14h: PMRMSCL – Persistent Memory Region Memory Space Control Lower", "3.1.4.28 Offset E18h: PMRMSCU – Persistent Memory Region Memory Space Control Upper"]
---

# Source page 96

NVM Express® Base Specification, Revision 2.1

74
 Offset E14h: PMRMSCL – Persistent Memory Region Memory Space Control Lower
This optional property and the PMRMSCU property specify  how the controller references the Persistent
Memory Region with host -supplied addresses. If the controller supports the Persistent Memory Region’s
controller memory space (PMRCAP.CMSS), this property is mandatory. Otherwise, this property is
reserved. The host shall access this register with aligned 32-bit accesses.
This property shall not be reset by Controller Reset.
Figure 63: Offset E14h: PMRMSCL – Persistent Memory Region Memory Space Control Lower
Bits Type Reset Description
31:12 RW 0h
Controller Base Address (CBA): This field specifies the 20 least significant bits of the
52 most significant bits of the 64 -bit base address for the Persistent Memory Region’s
controller address range. The Persistent Memory Region’s controller base address and
its size determine its controller address range.
The 64-bit base address specified by this field and PMRMSCU.CBA when the CMSE bit
is set to ‘1’ shall be valid only under the following conditions:
a) no part of the Persistent Memory Region’s controller address range is greater
than 264 − 1; and
b) if the Controller Memory Buffer’s controller memory space is enabled, then the
Persistent Memory Region’s controller address range does not overlap the
Controller Memory Buffer’s controller address range.
11:02 RO 0h Reserved
01 RW 0b
Controller Memory Space Enable (CMSE):  This bit specifies whether addresses
supplied by the host are permitted to reference the Persistent Memory Region.
If this bit is set to ‘1’ and the controller base address is valid, then the Persistent Memory
Region’s controller memory space is enabled. Otherwise, the controller memory space
is not enabled.
If the Persistent Memory Region’s controller memory space is enabled, then addresses
supplied by the host that fall within the Persistent Memory Region’s controller address
range shall reference the Persistent Memory Region.
If the Persistent Memory Region’s controller memory space is not enabled, then no
address supplied by the host shall reference the Persistent Memory Region. Instead,
such addresses shall reference memory spaces other than the Persistent Memory
Region.
00 RO 0b Reserved
 Offset E18h: PMRMSCU – Persistent Memory Region Memory Space Control Upper
This optional property and the PMRMSCL property specify how the controller references the Persistent
Memory Region with host -supplied addresses. If the controller supports the Persistent Memory Region’s
controller memory space (PMRCAP.CMSS), this property is mandatory. Otherwise, this property is
reserved. The host shall access this register with aligned 32-bit accesses.
This register shall not be reset by Controller Reset.
Figure 64: Offset E18h: PMRMSCU – Persistent Memory Region Memory Space Control Upper
Bits Type Reset Description
31:00 RW 0h
Controller Base Address (CBA): This field specifies the 32 most significant bits of the
52 most significant bits of the 64-bit base address for the Persistent Memory Region’s
controller address range. The Persistent Memory Region’s controller base address and
its size determine its controller address range.
