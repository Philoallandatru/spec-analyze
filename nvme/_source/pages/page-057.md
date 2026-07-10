---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 57
headings: ["2.4.2 Asymmetric Controller Behavior"]
---

# Source page 57

NVM Express® Base Specification, Revision 2.1

35
Figure 22: PCI Express Device Supporting Single Root I/O Virtualization (SR-IOV)

Examples provided in this section are meant to illustrate concepts and are not intended to enumerate all
possible configurations. For example, an NVM subsystem may contain multiple PCI Express ports with
each port supporting SR-IOV.
 Asymmetric Controller Behavior
Asymmetric controller behavior occurs in NVM subsystems where namespace access characteristics (e.g.,
performance) may vary based on:
• the internal configuration of the NVM subsystem; or
• which controller is used to access a namespace (e.g., Fabrics).
NVM subsystems that provide asymmetric controller behavior may support Asymmetric Namespace
Access Reporting as described in section 8.1.1.
Physical Function 0
NVMe Controller
Virtual Function (0,1)
NVMe Controller
Virtual Function (0,2)
NVMe Controller
Virtual Function (0,3)
NVMe Controller
Virtual Function (0,4)
NVMe Controller
NSID 1 NSID 2 NSID 3 NSID 2 NSID 4 NSID 2 NSID 5 NSID 2 NSID 6 NSID 2
NS
A
NS
B
NS
C
NS
D
NS
E
NS
F
PCIe Port
