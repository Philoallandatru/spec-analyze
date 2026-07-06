---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 626
headings: ["8.2.6.4 Single Root I/O Virtualization and Sharing (SR-IOV)", "8.3 Message-Based Transport Extended Capabilities (Fabrics)", "8.3.1 Automated Discovery of NVMe-oF Discovery Controllers for IP Based Fabrics"]
---

# Source page 626

NVM Express® Base Specification, Revision 2.1

604
A primary controller or secondary controller is enabled when CC.EN and CSTS.RDY are both set to ‘1’ for
that controller. A secondary controller is able to be enabled only when in the Online state. If the primary
controller associated with a secondary contr oller is disabled or undergoes a Controller Level Reset, then
the secondary controller shall implicitly transition to the Offline state. A secondary controller shall transition
to the Offline state when a shutdown occurs (refer to section 3.6, section 3.1.4.5, and section 3.1.4.20) on
the primary controller associated with that secondary controller.
Resources shall only be assigned to a secondary controller when in the Offline state. If the minimum number
of resources are not assigned to a secondary controller, then a request to transition to the Online state shall
fail for that secondary controller. For implementations that support SR-IOV, if VF Enable is cleared to ‘0’ or
NumVFs specifies a value that does not enable the associated secondary controller, then the secondary
controller shall implicitly transition to the Offline state.
 Single Root I/O Virtualization and Sharing (SR-IOV)
The PCI -SIG® PCI Express Base specification defines Single Root I/O Virtualization and Sharing
Specification (SR-IOV) extensions to PCI Express that allow multiple System Images (SIs), such as virtual
machines running on a hypervisor, to share PCI hardware resources. The primary benefit of SR-IOV is that
it eliminates the hypervisor from participating in I/O operations which may be a significant factor limiting
storage performance in some virtualized environments and allows direct SI access to PCI hardware
resources.
A Physical Function  (PF) is a PCI Express Function that supports the SR -IOV Capability, which in turn
allows that PF to support one or more dependent Virtual Functions (VFs). These PFs and VFs may support
NVM Express controllers that share an underlying NVM subsystem with mul ti-path I/O and namespace
sharing capabilities (refer to section 2.4.1).
SR-IOV Virtual Functions (VFs) with an NVM Express Class Code (refer to the PCI Header section of the
NVMe over PCIe Transport Specification) shall implement fully compliant NVM Express controllers. This
ensures that the same host software developed for no n-virtualized environments is capable of running
unmodified within an SI.
For hosts where SR-IOV is unsupported or not needed, a controller that is a PF shall support operation as
a stand-alone controller.
For a controller that is a PF, the requirements for SR -IOV Capability registers VF BAR0, VF BAR1, VF
BAR2, VF BAR4, and VF BAR5 are the same as the requirements for PCI registers BAR0, BAR1, BAR4,
and BAR5, respectively. For a controller that is a PF, SR-IOV Capability register VF BAR2 shall not support
Index/Data Pair. Refer to the PCI Header section of the NVMe over PCIe Transport Specification.
To accommodate SR -IOV address range isolation requirements, VF BAR2 and VF BAR3 may support a
64-bit prefetchable memory register space which shall only be used for MSI -X Tables and MSI-X PBAs of
VFs. MSI-X Table BIR = ‘2’ and MSI -X PBA BIR = ‘2’ are valid  for controllers that are VFs. Refer to the
MSI-X Capability section of the NVMe over PCIe Transport Specification.
While the controller properties of a controller that is a VF are accessible only if SR -IOV Control.VF MSE is
set to ‘1’, clearing VF MSE from ‘1’ to ‘0’ does not cause a reset of that controller. In this case, controller
properties are hidden, but their values are not reset.
8.3 Message-Based Transport Extended Capabilities (Fabrics)
This section describes extended capabilities that are specific to the Message-based transport model.
 Automated Discovery of NVMe-oF Discovery Controllers for IP Based Fabrics
When operating in an IP based fabric, before transmitting a Fabrics Connect command, the IP address of
the fabric interface of the Discovery controller is determined by one of the following methods:
a) administrative configuration;
b) discovered using DNS-SD (refer to RFC 6763); or
c) obtained by some means not defined in this specification.
