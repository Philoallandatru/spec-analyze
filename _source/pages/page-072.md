---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 72
headings: []
---

# Source page 72

NVM Express® Base Specification, Revision 2.1

50
Figure 33: Property Definition
Offset
(OFST)
Size
(in
bytes)
I/O
Controller1
Administrative
Controller1
Discovery
Controller1 Name
30h 8 M2 M2 R ACQ: Admin Completion Queue Base
Address
38h 4 O3 O3 R CMBLOC: Controller Memory Buffer
Location
3Ch 4 O3 O3 R CMBSZ: Controller Memory Buffer Size
40h 4 O3 O3 R BPINFO: Boot Partition Information
44h 4 O3 O3 R BPRSEL: Boot Partition Read Select
48h 8 O3 O3 R BPMBL: Boot Partition Memory Buffer
Location
50h 8 O3 O3 R CMBMSC: Controller Memory Buffer
Memory Space Control
58h 4 O3 O3 R CMBSTS: Controller Memory Buffer
Status
5Ch 4 O3 O3 R CMBEBS: Controller Memory Buffer
Elasticity Buffer Size
60h 4 O3 O3 R CMBSWTP: Controller Memory Buffer
Sustained Write Throughput
64h 4 O O R NSSD: NVM Subsystem Shutdown
68h 4 M M R CRTO: Controller Ready Timeouts
6Ch  R R R Reserved
E00h 4 O3 O3 R PMRCAP: Persistent Memory Capabilities
E04h 4 O3 O3 R PMRCTL: Persistent Memory Region
Control
E08h 4 O3 O3 R PMRSTS: Persistent Memory Region
Status
E0Ch 4 O3 O3 R PMREBS: Persistent Memory Region
Elasticity Buffer Size
E10h 4 O3 O3 R PMRSWTP: Persistent Memory Region
Sustained Write Throughput
E14h 4 O3 O3 R PMRMSCL: Persistent Memory Region
Controller Memory Space Control Lower
E18h 4 O3 O3 R PMRMSCU: Persistent Memory Region
Controller Memory Space Control Upper
E1Ch  R R R Reserved
1000h
Transport Specific:
• Refer to Figure 34 for Memory-Based transport implementations.
• Refer to Figure 35 for Message-Based transport implementations.
Notes:
1. O/M/R definition: O = Optional, M = Mandatory, R = Reserved
2. Mandatory for memory-based controllers. For message-based controllers this property is reserved.
3. Optional for memory-based controllers. For message-based controllers this property is reserved.
4. Determined by the transport (e.g., the offset calculation formula Offset (1000h + ((2y) * (4 << CAP.DSTRD))) for
the memory-based PCIe transport).
