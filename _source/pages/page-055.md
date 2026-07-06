---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 55
headings: []
---

# Source page 55

NVM Express® Base Specification, Revision 2.1

33
namespace A and NSID 2 is associated with namespace B. Both namespaces are private to the controller
and this configuration supports neither multi-path I/O nor namespace sharing.
Figure 19: NVM Express Controller with Two Namespaces
NSID 1 NSID 2
PCI Function 0
NVM Express Controller
PCIe Port
NS
A
NS
B

Figure 20 shows a multi-Function NVM subsystem with a single PCI Express port containing two controllers
implementing NVMe over PCIe. One controller is associated with PCI Function 0 and the other controller is
associated with PCI Function 1. Each controller supports a single private namespace and access to shared
namespace B. The namespace ID shall be the same in all controllers that have access to a particular shared
namespace. In this example, both controllers use NSID 2 to access shared namespace B.
Figure 20: NVM Subsystem with Two Controllers and One Port

There is one or more Identify Controller data structure s for each controller and one or more Identify
Namespace data structures (refer to section 1.5.49) for each namespace (refer to Figure 310). Controllers
with access to a shared namespace return the Identify Namespace data structure associated with that
shared namespace (i.e., the same data structure contents are returned by all controllers with access to the
same shared namespace). There is a  globally unique identifier (refer to section 4.7.1) associated with the
namespace itself and may be used to determine when there are multiple paths to the same shared
namespace.
Controllers associated with a shared namespace may operate on the namespace concurrently. Operations
performed by individual controllers are atomic to the shared namespace at the write atomicity level of the
NSID 1 NSID 2
PCI Function 0
NVMe Controller
PCIe Port
NS
A
NS
B
NSID 3 NSID 2
PCI Function 1
NVMe Controller
NS
C
