---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 56
headings: []
---

# Source page 56

NVM Express® Base Specification, Revision 2.1

34
controller to which the command was submitted (refer to section  3.4.3). The write atomicity level is not
required to be the same across controllers that share a namespace. If there are any ordering requirements
between commands issued to different controllers that access a shared namespace, then host software or
an associated application, is required to enforce these ordering requirements.
Figure 21 illustrates an NVM subsystem with two PCI Express ports, each with an associated controller
implementing NVMe over PCIe . Both controllers map to PCI Function 0 of the corresponding port. Each
PCI Express port in this example is completely independent and has its own PCI Express Fundamental
Reset and reference clock input. A reset of a port only affects the controller assoc iated with that port and
has no impact on the other controller, shared namespace, or operations performed by the other controller
on the shared namespace. Refer to section 4.4 for Feature behavior on reset. The functional behavior of
this example is otherwise the same as that illustrated in Figure 20.
Figure 21: NVM Subsystem with Two Controllers and Two Ports
NSID 1 NSID 2
PCI Function 0
NVMe Controller
NS
A
NS
B
NSID 3 NSID 2
PCI Function 0
NVMe Controller
NS
C
PCIe Port x PCIe Port y

The two ports shown in Figure 21 may be associated with the same Root Complex or with different Root
Complexes and may be used to implement both multi-path I/O and I/O sharing architectures. System-level
architectural aspects and use of multiple ports in a PCI Express fabric are beyond t he scope of this
specification.
Figure 22 illustrates an NVM subsystem that supports Single Root I/O Virtualization (SR-IOV) and has one
Physical Function and four Virtual Functions. An NVM Express controller implementing NVMe over PCIe is
associated with each Function with each controller having a private namespace and access to a namespace
shared by all controllers, labeled NS F. The behavior of the controllers in this example parallels that of the
other examples in this section. Refer to section 8.2.6.4 for more information on SR-IOV.
