---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 34
headings: ["1.5.72 physical fabric interface (physical ports)", "1.5.73 Placement Handle", "1.5.74 Placement Identifier", "1.5.75 Port ID", "1.5.76 Ports List", "1.5.77 Power Loss Acknowledge (PLA)", "1.5.78 Power Loss Notification (PLN)", "1.5.79 primary controller", "1.5.80 private namespace", "1.5.81 property", "1.5.82 Reclaim Group (RG)", "1.5.83 Reclaim Unit (RU)", "1.5.84 Reclaim Unit Handle (RUH)"]
---

# Source page 34

NVM Express® Base Specification, Revision 2.1

12
 physical fabric interface (physical ports)
A physical connection between an NVM subsystem and a fabric.
 Placement Handle
A namespace scoped handle that maps to an Endurance Group scoped Reclaim Unit Handle which
references a Reclaim Unit in each Reclaim Group.
 Placement Identifier
A data structure that specifies a Reclaim Group Identifier and a Placement Handle that references a
Reclaim Unit. Refer to Figure 282 and Figure 283.
 Port ID
An identifier that is associated with an NVM subsystem port. Refer to section 2.2.2.
 Ports List
A list of ports that may be used to export an NVM subsystem. Entries in the Ports List are in the format
specified by Underlying Fabrics Transport Entry data structure (refer to Figure 336).
 Power Loss Acknowledge (PLA)
The transport-specific variable that is used by the controller to inform the host of the controller’s current
Power Loss Signaling processing (refer to section 8.2.5).
 Power Loss Notification (PLN)
The transport-specific variable that is used to inform the controller that a main power loss event is expected
to occur (refer to section 8.2.5).
 primary controller
An NVM Express controller that supports the Virtualization Management command. An NVM subsystem
may contain multiple primary controllers. Secondary controller(s) in an NVM subsystem depend on a
primary controller for dynamic resource management (refer to section 8.2.6).
A PCI Express SR -IOV Physical Function that supports the NVM Express interface and the Virtualization
Enhancements capability is an example of a primary controller (refer to section 8.2.6.4).
 private namespace
A namespace that is only able to be attached to one controller at a time . Refer to the Namespace Multi-
path I/O and Namespace Sharing Capabilities (NMIC) field in Figure 319.
 property
The generalization of memory mapped controller registers defined for NVMe over PCIe. Properties are
used to configure low level controller attributes and obtain low level controller status. Refer to section 3.1.4.
 Reclaim Group (RG)
An entity that contains one or more Reclaim Units. Refer to section 3.2.4.
 Reclaim Unit (RU)
A logical representation of non-volatile storage within a Reclaim Group that is able to be physically erased
by the controller without disturbing any other Reclaim Units. Refer to section 3.2.4.
 Reclaim Unit Handle (RUH)
A controller resource that references a Reclaim Unit in each Reclaim Group. Refer to section 3.2.4.
