---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 10
headings: []
---

# Source page 10

NVM Express® Base Specification, Revision 2.1

x
Table of Figures
Figure 1: NVMe Family of Specifications ....................................................................................................................... 1
Figure 2: Decimal and Binary Units ................................................................................................................................ 3
Figure 3: Byte, Word, and Dword Relationships ............................................................................................................ 5
Figure 4: Taxonomy of Transport Models .................................................................................................................... 18
Figure 5: Types of NVMe Command Sets .................................................................................................................... 19
Figure 6: Queue Pair Example, 1:1 Mapping ............................................................................................................... 20
Figure 7: Queue Pair Example, n:1 Mapping ............................................................................................................... 20
Figure 8: NVMe over Fabrics Layering ......................................................................................................................... 22
Figure 9: Command Capsule Format ........................................................................................................................... 23
Figure 10: Response Capsule Format ......................................................................................................................... 24
Figure 11: Simple NVM Storage Hierarchy with NVM Sets .......................................................................................... 25
Figure 12: Simple NVM Storage Hierarchy with One Reclaim Group .......................................................................... 26
Figure 13: Simple NVM Storage Hierarchy with Multiple Reclaim Groups ................................................................... 27
Figure 14: Complex NVM Storage Hierarchy with NVM Sets ....................................................................................... 28
Figure 15: Complex NVM Storage Hierarchy with Multiple Reclaim Groups ................................................................ 29
Figure 16: Single-Namespace NVM Subsystem .......................................................................................................... 30
Figure 17: Two-Namespace NVM Subsystem ............................................................................................................. 31
Figure 18: Complex NVM Subsystem .......................................................................................................................... 32
Figure 19: NVM Express Controller with Two Namespaces ......................................................................................... 33
Figure 20: NVM Subsystem with Two Controllers and One Port .................................................................................. 33
Figure 21: NVM Subsystem with Two Controllers and Two Ports ................................................................................ 34
Figure 22: PCI Express Device Supporting Single Root I/O Virtualization (SR-IOV) ................................................... 35
Figure 23: Controller Types .......................................................................................................................................... 37
Figure 24: NVM Subsystem with Three I/O Controllers ................................................................................................ 38
Figure 25: NVM Subsystem with One Administrative and Two I/O Controllers ............................................................ 40
Figure 26: NVM Subsystem with One Administrative Controller .................................................................................. 40
Figure 27: Controller IDs FFF0h to FFFFh ................................................................................................................... 42
Figure 28: Admin Command Support Requirements .................................................................................................... 43
Figure 29: Fabrics Command Support Requirements .................................................................................................. 44
Figure 30: Common I/O Command Support Requirements ......................................................................................... 45
Figure 31: Log Page Support Requirements ................................................................................................................ 45
Figure 32: Feature Support Requirements ................................................................................................................... 47
Figure 33: Property Definition ...................................................................................................................................... 49
Figure 34: Memory-Based Property Definition ............................................................................................................. 51
Figure 35: Message-Based Property Definition ............................................................................................................ 51
Figure 36: Offset 0h: CAP – Controller Capabilities ..................................................................................................... 52
Figure 37: Specification Version Descriptor ................................................................................................................. 55
Figure 38: NVM Express Base Specification Version Property Reset Values .............................................................. 55
Figure 39: Offset Ch: INTMS – Interrupt Mask Set....................................................................................................... 56
Figure 40: Offset 10h: INTMC – Interrupt Mask Clear .................................................................................................. 56
Figure 41: Offset 14h: CC – Controller Configuration ................................................................................................... 57
Figure 42: Offset 1Ch: CSTS – Controller Status ......................................................................................................... 60
Figure 43: Offset 20h: NSSR – NVM Subsystem Reset ............................................................................................... 63
Figure 44: Offset 24h: AQA – Admin Queue Attributes ................................................................................................ 63
Figure 45: Offset 28h: ASQ – Admin Submission Queue Base Address ..................................................................... 63
Figure 46: Offset 30h: ACQ – Admin Completion Queue Base Address ...................................................................... 64
Figure 47: Offset 38h: CMBLOC – Controller Memory Buffer Location ........................................................................ 64
Figure 48: Offset 3Ch: CMBSZ – Controller Memory Buffer Size................................................................................. 65
Figure 49: Offset 40h: BPINFO – Boot Partition Information ........................................................................................ 66
Figure 50: Offset 44h: BPRSEL – Boot Partition Read Select ..................................................................................... 66
Figure 51: Offset 48h: BPMBL – Boot Partition Memory Buffer Location ..................................................................... 67
Figure 52: Offset 50h: CMBMSC – Controller Memory Buffer Memory Space Control ................................................ 67
Figure 53: Offset 58h: CMBSTS – Controller Memory Buffer Status............................................................................ 68
Figure 54: Offset 5Ch: CMBEBS – Controller Memory Buffer Elasticity Buffer Size .................................................... 68
Figure 55: Offset 60h: CMBSWTP – Controller Memory Buffer Sustained Write Throughput ...................................... 68
Figure 56: Offset 64h: NSSD – NVM Subsystem Shutdown ........................................................................................ 69
Figure 57: Offset 68h: CRTO – Controller Ready Timeouts ......................................................................................... 70
Figure 58: Offset E00h: PMRCAP – Persistent Memory Region Capabilities .............................................................. 70
Figure 59: Offset E04h: PMRCTL – Persistent Memory Region Control ...................................................................... 71
Figure 60: Offset E08h: PMRSTS – Persistent Memory Region Status ....................................................................... 72
Figure 61: Offset E0Ch: PMREBS – Persistent Memory Region Elasticity Buffer Size ................................................ 73
