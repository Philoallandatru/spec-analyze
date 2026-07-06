---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 90
headings: ["3.1.4.17 Offset 58h: CMBSTS – Controller Memory Buffer Status", "3.1.4.18 Offset 5Ch: CMBEBS – Controller Memory Buffer Elasticity Buffer Size", "3.1.4.19 Offset 60h: CMBSWTP – Controller Memory Buffer Sustained Write Throughput"]
---

# Source page 90

NVM Express® Base Specification, Revision 2.1

68
 Offset 58h: CMBSTS – Controller Memory Buffer Status
This optional property indicates the status of the Controller Memory Buffer. If the controller supports the
Controller Memory Buffer (CAP.CMBS), this property is mandatory. Otherwise, this property is reserved.
Figure 53: Offset 58h: CMBSTS – Controller Memory Buffer Status
Bits Type Reset Description
31:01 RO 0h Reserved
00 RO 0b
Controller Base Address Invalid (CBAI): This bit indicates whether the controller has
failed to enable the Controller Memory Buffer’s controller memory space because
CMBMSC.CBA is invalid. If CMBMSC.CRE and CMBMSC.CMSE are set to ‘1’, and
CMBMSC.CBA is invalid, this bit shall be set to ‘1’. Otherwise, this bit shall be cleared to
‘0’.
 Offset 5Ch: CMBEBS – Controller Memory Buffer Elasticity Buffer Size
This optional property identifies to the host the size of the CMB elasticity buffer. A value of 0h in this property
indicates to the host that no information regarding the presence or size of a CMB elasticity buffer is
available.
Figure 54: Offset 5Ch: CMBEBS – Controller Memory Buffer Elasticity Buffer Size
Bits Type Reset Description
31:8 RO Impl
Spec
CMB Elasticity Buffer Size Base (CMBWBZ): Indicates the size of the CMB elasticity
buffer. The size of the CMB elasticity buffer is equal to the value in this field multiplied
by the value specified by the CMB Elasticity Buffer Size Units field.
7:5 RO 0h Reserved
4 RO Impl
Spec
CMB Read Bypass Behavior (CMBRBB): If a memory read does not conflict with any
memory write in the CMB Elasticity Buffer (i.e., if the set of memory addresses specified
by a read is disjoint from the set of memory addresses specified by all writes in the
CMB Elasticity Buffer), and this bit is:
a) set to ‘1’, then memory reads not conflicting with memory writes in the CMB
Elasticity Buffer shall bypass those memory writes; and
b) cleared to ‘0’, then memory reads not conflicting with memory writes in the
CMB Elasticity Buffer may bypass those memory writes.
3:0 RO Impl
Spec
CMB Elasticity Buffer Size Units (CMBSZU):  Indicates the granularity of the CMB
Elasticity Buffer Size Base field.
Value Granularity
0h Bytes
1h 1 KiB
2h 1 MiB
3h 1 GiB
4h – Fh Reserved

 Offset 60h: CMBSWTP – Controller Memory Buffer Sustained Write Throughput
This optional property identifies to the host the maximum CMB sustained write throughput. A value of 0h in
this property indicates to the host that no information regarding the CMB sustained write throughput is
available.
Figure 55: Offset 60h: CMBSWTP – Controller Memory Buffer Sustained Write Throughput
Bits Type Reset Description
31:8 RO Impl
Spec
CMB Sustained Write Throughput (CMBSWTV): Indicates the sustained write
throughput of the CMB at the maximum  payload size specified by the applicable NVMe
Transport binding specification (e.g. , the PCIe TLP payload size, as specified in the
Max_Payload_Size (MPS) field of the PCIe Express Device Control (PXDC) register ).
The sustained write throughput of the CMB is equal to the value in this field multiplied by
the units specified by the CMB Sustained Write Throughput Units field.
