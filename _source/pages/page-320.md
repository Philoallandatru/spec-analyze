---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 320
headings: []
---

# Source page 320

NVM Express® Base Specification, Revision 2.1

298
Figure 310: Identify – CNS Values
CNS
Value O/M1 Definition NSID2 CNTID3 CSI4 Reference
Section
18h O10 Domain List N N N 5.1.13.2.15
19h O9 Endurance Group List N N N 5.1.13.2.16
1Ah O5 I/O Command Set specific Allocated Namespace ID
list Y N Y 5.1.13.2.17
1Bh O5 I/O Command Set specific Identify Namespace data
structure for the specified allocated NSID. Y N Y 5.1.13.2.18
1Ch O I/O Command Set data structure N Y N 5.1.13.2.19
1Dh O Get Underlying Namespace List12 Y Y N 5.1.13.4.1
1Eh O Get Ports List N N N 5.1.13.4.2
1Fh O13 I/O Command Set Independent Identify Namespace
data structure for the specified allocated NSID. Y N N 5.1.13.2.20
20h O Supported Controller State Formats N N N 5.1.13.2.21
Future Definition
21h to FFh  Reserved
Notes:
1. O/M definition: O = Optional, M = Mandatory.
2. The NSID field is used: Y = Yes, N = No.
3. The CDW10.CNTID field is used: Y = Yes, N = No.
4. The CDW11.CSI field is used: Y = Yes, N = No.
5. Mandatory for controllers that support the Namespace Management capability (refer to section 8.1.15).
6. Mandatory for controllers that support Virtualization Enhancements (refer to section 8.2.6).
7. Selection of a UUID may be supported (refer to section 8.1.28).
8. This Identify data structure applies to namespaces that are associated with command sets that specify logical blocks ( e.g.,
Command Set Identifier 0h or Command Set Identifier 2h).
9. Mandatory for controllers that support Variable Capacity Management (refer to section 8.1.4.3).
10. Mandatory for controllers that support Capacity Management (refer to section 8.1.4) in an NVM subsystem that supports
multiple domains (refer to section 3.2.5).
11. Only applicable for the NVM Command Set and I/O Command Sets based on the NVM Command Set. Prohibited for all other
I/O Command Sets.
12. Support for this CNS value is prohibited in NVM subsystems that use a Memory -Based Transport Model (e.g., the PCIe
transport) for any controller.
13. For controllers compliant with NVM Express Base Specification, Revision TBD and later, mandatory if the Namespace
Management capability is supported.
The Command Set Identifier values are defined in Figure 311.
Figure 311: Command Set Identifiers
Command
Set Identifier
Value
Definition Reference
00h NVM Command Set Refer to the NVM Command Set Specification
01h Key Value Command Set Refer to the Key Value Command Set Specification
02h Zoned Namespace Command Set Refer to the Zoned Namespace Command Set Specification
03h Subsystem Local Memory Command Set Refer to the Subsystem Local Memory Command Set Specification
04h Computational Programs Command Set Refer to the Computational Programs Command Set Specification
05h to 2Fh Reserved
30h to 3Fh Vendor specific
40h to FFh Reserved
