---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 95
headings: ["3.1.4.25 Offset E0Ch: PMREBS – Persistent Memory Region Elasticity Buffer Size", "3.1.4.26 Offset E10h: PMRSWTP – Persistent Memory Region Sustained Write Throughput"]
---

# Source page 95

NVM Express® Base Specification, Revision 2.1

73
 Offset E0Ch: PMREBS – Persistent Memory Region Elasticity Buffer Size
This optional property identifies to the host the size of the PMR elasticity buffer. A value of 0h in this property
indicates to the host that no information regarding the presence or size of a PMR elasticity buffer is
available.
This property shall not be reset by Controller Reset.
Figure 61: Offset E0Ch: PMREBS – Persistent Memory Region Elasticity Buffer Size
Bits Type Reset Description
31:8 RO Impl
Spec
PMR Elasticity Buffer Size Base (PMRWBZ):  Indicates the size of the PMR elasticity
buffer. The actual size of the PMR elasticity buffer is equal to the value in this field
multiplied by the value specified by the PMR Elasticity Buffer Size Units field.
7:5 RO 000b Reserved
4 RO Impl
Spec
PMR Read Bypass Behavior (PMRRBB): If a memory read does not conflict with any
memory write in the PMR Elasticity Buffer (i.e., if the set of memory addresses specified
by a read is disjoint from the set of memory addresses specified by all writes in the PMR
Elasticity Buffer), and this bit is:
a) set to ‘1’, then memory reads not conflicting with memory writes in the PMR
Elasticity Buffer shall bypass those memory writes; and
b) cleared to ‘0’, then memory reads not conflicting with memory writes in the PMR
Elasticity Buffer may bypass those memory writes.
3:0 RO Impl
Spec
PMR Elasticity Buffer Size Units (PMRSZU):  Indicates the granularity of the PMR
Elasticity Buffer Size Base field.
Value Definition
0h Bytes
1h 1 KiB
2h 1 MiB
3h 1 GiB
4h to Fh Reserved

 Offset E10h: PMRSWTP – Persistent Memory Region Sustained Write Throughput
This optional property identifies to the host the maximum PMR sustained write throughput. A value of 0h in
this property indicates to the host that no information regarding the PMR sustained write throughput is
available.
This property shall not be reset by Controller Reset.
Figure 62: Offset E10h: PMRSWTP – Persistent Memory Region Sustained Write Throughput
Bits Type Reset Description
31:8 RO Impl
Spec
PMR Sustained Write Throughput (PMRSWTV):  Indicates the sustained write
throughput of the PMR at the maximum payload size specified by the applicable NVMe
Transport binding specification (e.g. , the PCIe TLP payload size, as specified in the
Max_Payload_Size (MPS) field of the PCI Express Device Control (PXDC) register). The
actual sustained write throughput of the PMR is equal to the value in this field multiplied
by the units specified by the PMR Sustained Write Throughput Units field.
7:4 RO 0h Reserved
3:0 RO Impl
Spec
PMR Sustained Write Throughput Units (PMRSWTU): Indicates the granularity of the
PMR Sustained Write Throughput field.
Value Definition
0h Bytes per second
1h 1 KiB / s
2h 1 MiB / s
3h 1 GiB / s
7h to Fh Reserved
