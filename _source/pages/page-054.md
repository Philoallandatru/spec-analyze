---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 54
headings: ["2.4 Extended Capabilities Theory", "2.4.1 Multi-Path I/O and Namespace Sharing"]
---

# Source page 54

NVM Express® Base Specification, Revision 2.1

32
An NVM subsystem may have multiple domains, multiple namespaces, multiple controllers, and multiple
ports, as shown in Figure 18.
Figure 18: Complex NVM Subsystem

2.4 Extended Capabilities Theory
 Multi-Path I/O and Namespace Sharing
This section provides an overview of multi-path I/O and namespace sharing. Multi-path I/O refers to two or
more completely independent paths between a single host and a namespace while namespace sharing
refers to the ability for two or more hosts to access  a common shared namespace using different NVM
Express controllers. Both multi-path I/O and namespace sharing require that the NVM subsystem contain
two or more controllers. NVM subsystems that support Multi -Path I/O and Namespace Sharing may also
support asymmetric controller behavior (refer to section 2.4.2). Concurrent access to a shared namespace
by two or more hosts requires some form of coordination between hosts. The procedure used to coordinate
these hosts is outside the scope of this specification.
Figure 19 shows an NVM subsystem that contains a single NVM Express controller implemented over PCI
Express and a single PCI Express port. Since this is a single Function PCI Express device, the NVM
Express controller shall be associated with PCI Function 0. A controller may support multiple namespaces.
The controller in Figure 19 supports two namespaces labeled NS A and NS B. Associated with each
controller namespace is a namespace ID, labeled as NSID 1 and NSID 2, that is used by the controller to
reference a specific namespace. The namespace ID is distinct from the namespace itself and is the handle
a host and controller use to specify a particular namespace in a command. The selection of a controller’s
namespace IDs is outside the scope of this specification. In this example , NSID 1 is associated with
NVM Subsystem
Domain 2
Endurance Group 3
NVM Set 3
Endurance Group 2
NVM Set 2
Domain 1
Endurance Group 1
NVM Set 1
Port
Controller
Namespace 1
Port
Controller
Namespace 2
Port
Controller
Namespace 3
...
Namespace 4
