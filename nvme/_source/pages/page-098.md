---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 98
headings: ["3.2.1.6 NSID and Namespace Usage"]
---

# Source page 98

NVM Express® Base Specification, Revision 2.1

76
Figure 65: NSID Types and Relationship to Namespace
Valid NSID Type NSID relationship to namespace Reference
Allocated Refers to a namespace that exists in the NVM subsystem 3.2.1.3
Inactive Does not refer to a namespace that is attached to the controller1 3.2.1.4
Active Refers to a namespace that is attached to the controller 3.2.1.4
Notes:
1. If allocated, refers to a namespace that is not attached to the controller. If unallocated, does not refer to any
namespace.
Figure 66: NSID Types
Inv. Valid Invalid
0 1 NN NN+1
NSID
Allocated Unallocated
Active Inactive
NVM
Subsystem
Controller
FFFFFFFFh
B
Broadcast Value

 NSID and Namespace Usage
If Namespace Management (refer to section 8.1.15), ANA Reporting (refer to section 8.1.1), or NVM Sets
(refer to section 3.2.2) capabilities are supported, then NSIDs shall be unique within the NVM subsystem
(e.g., NSID of 3 shall refer to the same physical namespace regardless of the accessing controller). If the
Namespace Management, ANA Reporting, and NVM Sets capabilities are not supported, then NSIDs:
a) for shared namespaces shall be unique within the NVM subsystem; and
b) for private namespaces are not required to be unique within the NVM subsystem.
The Identify command (refer to section 5.1.13) may be used to determine the active NSIDs for a controller
and the allocated NSIDs in the NVM subsystem.
If the MNAN field (refer to Figure 312) is cleared to 0h, then the maximum number of allocated NSIDs is
the same as the value reported in the NN field (refer to Figure 312). If the MNAN field is non-zero, then the
maximum number of allocated NSIDs may be less than the number of namespaces (e.g., an NVM
subsystem may support a maximum valid NSID value (i.e., the NN field) set to 1,000,000 but support a
maximum of 10 allocated NSID values).
To determine the active NSIDs for a particular controller, the host may follow either of the following methods:
