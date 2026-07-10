---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 23
headings: ["1 Introduction", "1.1 Overview", "1.1.1 NVM Express® Specification Family"]
---

# Source page 23

1
1 Introduction
1.1 Overview
The NVM Express® (NVMe®) interface allows host software to communicate with a non -volatile memory
subsystem (NVM subsystem). This interface is optimized for all storage solutions, attached using a variety
of transports including PCI Express ®, Ethernet, InfiniBand TM, and Fibre Channel. The mapping of
extensions defined in this document to a specific NVMe Transport are defined in an NVMe Transport
binding specification. The NVMe Transport binding specification for Fibre Channel is defined in INCITS 556
Fibre Channel – Non-Volatile Memory Express - 2 (FC-NVMe-2).
For an overview of changes from revision 2.0 to revision 2.1, refer to https://nvmexpress.org/changes for a
document that describes the new features, including mandatory requirements for a controller to comply with
revision 2.1.
 NVM Express® Specification Family
Figure 1 shows the relationship of the NVM Express specifications to each other within the NVM e® family
of specifications.
Figure 1: NVMe Family of Specifications
I/O Command Set Specifications
(e.g., NVM, Key Value, Zoned Namespace)
Transport Specifications
(e.g., PCIe ®, RDMA, TCP)
NVM Express Base
Specification
NVM Express
Management Interface
Specification
Boot Specification

The NVM Express specification family structure shown in Figure 1 is intended to show the applicability of
NVM Express specifications to each other, not a hierarchy, protocol stack, or system architecture.
The NVM Express Base Specification (i.e., this specification)  defines a protocol for host software to
communicate with an NVM subsystem over a variety of memory -based transports and message -based
transports.
The NVM Express Management Interface (NVMe -MI) Specification defines an optional management
interface for all NVM Express Subsystems.
NVM Express I/O Command Set specifications define data structures, features, log pages, commands, and
status values that extend the NVM Express Base Specification.
NVM Express Transport specifications define the binding of the NVMe protocol  including controller
properties to a specific transport.
The NVM Express Boot Specification defines constructs and guidelines for booting from NVM Express
interfaces.
