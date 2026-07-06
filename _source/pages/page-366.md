---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 366
headings: ["5.1.13.3 Memory-Based Transport Identify Data Structures (PCIe)", "5.1.13.3.1 Primary Controller Capabilities data structure (CNS 14h)"]
---

# Source page 366

NVM Express® Base Specification, Revision 2.1

344
Figure 329: Supported Controller State Formats Data Structure
Bytes Description
NVMe Controller State Version list
3:2
NVMe Controller State Data Structure Version 1: This field contains a version of the NVMe
Controller State data structure  (refer to Version field in  Figure 359) that is supported by this
controller and is associated with index 1h of this list, if any. This field is the first entry of the
NVMe Controller State Version list.
5:4
NVMe Controller State Data Structure Version 2: This field contains a version of the NVMe
Controller State data structure that is supported by this controller and is associated with index
2h of this list, if any. This field is the second entry of the NVMe Controller State Version list.
…
(NV*2)+1:(NV*2)
NVMe Controller State Data Structure Version NV: This field contains a version of the NVMe
Controller State data structure that is supported by this controller and is associated with index
NV of this list, if any. This field is the last entry of the NVMe Controller State Version list.
Vendor Specific Controller State UUID Supported List
((NV+1)*2)+15:
((NV+1)*2)
Vendor Specific Controller State UUID 1: This field contains a  128-bit Universally Unique
Identifier (UUID), as specified in RFC 9562 (refer to section 4.5.6), that identifies the format of
the Vendor Specific Data field the Controller State data structure  (refer to Figure 358) that is
supported by this controller and is associated with index 1h of this list, if any. This field is the
first entry of the Vendor Specific Controller State UUID Supported list.
The value in this field and the associated format of the Vendor Specific Data field the Controller
State data structure is out of scope for this specification.
((NV+1)*2)+31 :
((NV+1)*2)+16
Vendor Specific Controller State UUID 2: This field contains a  128-bit Universally Unique
Identifier (UUID), as specified in RFC 9562, that identifies the format of the Vendor Specific
Data field the Controller State data structure  that is supported by this controller and is
associated with index 2h of this list, if any. This field is the second entry of the Vendor Specific
Controller State UUID Supported list.
The value in this field and the associated format of the Vendor Specific Data field the Controller
State data structure is out of scope for this specification.
…
((NUUID)*16)+
 ((NV+1)*2)-1 :
((NUUID-1)*16)+
((NV+1)*2)
Vendor Specific Controller State UUID NUUID: This field contains a  128-bit Universally
Unique Identifier (UUID), as specified in RFC 9562, that identifies the format of the Vendor
Specific Data field the Controller State data structure that is supported by this controller and is
associated with index NUUID of this list, if any. This field is the last entry of the Vendor Specific
Controller State UUID Supported list.
The value in this field and the associated format of the Vendor Specific Data field the Controller
State data structure is out of scope for this specification.
 Memory-Based Transport Identify Data Structures (PCIe)
This section describes Identify data structures that are specific to the Memory -based transport model.
5.1.13.3.1 Primary Controller Capabilities data structure (CNS 14h)
The Primary Controller Capabilities Structure (refer to Figure 330) is returned to the host for the primary
controller specified.
Figure 330: Identify – Primary Controller Capabilities Structure
Bytes Description
01:00 Controller Identifier (CNTLID): This field indicates the Controller Identifier of the primary controller.
03:02
Port Identifier (PORTID): This field indicates the Port Identifier of the NVM subsystem port associated
with the primary controller. The Port Identifier for a PCI Express Port shall be unique within the NVM
subsystem.
If the NVM subsystem supports an NVMe -MI Management Endpoint on this PCIe port, then this field
shall contain the same value as the Port Identifier field in the Controller Information Data Structure (refer
to the NVM Express Management Interface Specification) for this primary controller.
