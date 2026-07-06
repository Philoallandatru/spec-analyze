---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 368
headings: ["5.1.13.4 Message-Based Transport Identify Data Structures (Fabrics)", "5.1.13.4.1 Get Underlying Namespace List (CNS 1Dh)"]
---

# Source page 368

NVM Express® Base Specification, Revision 2.1

346
Figure 331: Secondary Controller List
Bytes Description
00
Number of Entries (NUMENT): This field indicates the number of Secondary Controller
Entries in the list. There are up to 127 entries in the list. A value of 0h indicates there are
no entries in the list.
31:01 Reserved
Secondary Controller Entry List
63:32 SC Entry 0: This field contains the first Secondary Controller Entry in the list, if present.
95:64 SC Entry 1:  This field contains the second Secondary Controller Entry in the list, if
present.
… …
(NUMENT-1)*32+63):
((NUMENT-1)*32+32)
SC Entry NUMENT-1: This field contains the last Secondary Controller Entry in the list,
if present.

Figure 332: Secondary Controller Entry
Bytes Description
01:00 Secondary Controller Identifier (SCID):  This field indicates the Controller Identifier of the secondary
controller described by this entry.
03:02 Primary Controller Identifier (PCID): This field indicates the Controller Identifier of the associated
primary controller.
04
Secondary Controller State (SCS): This field indicates the state of the secondary controller.
Bits Description
7:1 Reserved
0 Online State (OLS): If this bit is set to ‘1’, then the controller is in the Online state. If this bit
is cleared to ‘0’, then the controller is in the Offline state.

07:05 Reserved
09:08
Virtual Function Number (VFN): If the secondary controller is an SR -IOV VF, this field indicates its VF
Number, where VF Number > 0, and VF Number is no larger than the total number of VFs indicated by the
TotalVFs register (refer to Single Root I/O Virtualization and Sharing Specification) in the PF’s SR -IOV
Extended Capability structure. If the secondary controller is not an SR-IOV VF, then this field is cleared to
0h.
11:10 Number of VQ Flexible Resources Assigned (NVQ):  This field indicates the number of VQ Flexible
Resources currently assigned to the indicated secondary controller.
13:12 Number of VI Flexible Resources Assigned (NVI):  This field indicates the number of VI Flexible
Resources currently assigned to the indicated secondary controller.
31:14 Reserved
 Message-Based Transport Identify Data Structures (Fabrics)
This section describes Identify data structures that are specific to the Message -based transport model.
5.1.13.4.1 Get Underlying Namespace List (CNS 1Dh)
An Underlying Namespace List data structure (refer to Figure 333) is returned containing a list of all
Underlying Namespaces spanning all NVM subsystems that are accessible through either a virtual function
or a physical function.
Figure 333: Underlying Namespace List Data Structure
Bytes Description
07:00
Generation Counter (GENCTR): This field indicates the version of Underlying Namespace
information. This field shall be cleared to 0h as a result of NVM Subsystem Reset of the
Underlying NVM Subsystem. For each change in the Underlying Namespace List, this
counter is incremented by one. If the value of this field is FFFFFFFF_FFFFFFFFh, then the
field shall be cleared to 0h when incremented (i.e., rolls over to 0h).
15:08 Number Entries (NUMENT): Number of Underlying Namespace Entries.
