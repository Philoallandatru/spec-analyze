---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 30
headings: ["1.5.24 Discovery controller", "1.5.25 discovery information", "1.5.26 Discovery Service", "1.5.27 dispersed namespace", "1.5.28 dynamic controller", "1.5.29 Domain", "1.5.30 embedded management controller", "1.5.31 emulated controller", "1.5.32 Endurance Group", "1.5.33 Entry Key", "1.5.34 Exported Namespace", "1.5.35 Exported NVM Resources"]
---

# Source page 30

NVM Express® Base Specification, Revision 2.1

8
 Discovery controller
A controller that exposes capabilities that allow a host to retrieve a Discovery Log Page. A Discovery
controller does not implement I/O Queues or provide access to a non -volatile storage medium. Refer to
section 3.1.3.3.
 discovery information
Information about a host or NVM subsystem that is used for discovery (e.g., NVMe Transport address,
NQN, etc.).
 Discovery Service
An NVM subsystem that supports Discovery controllers only. A Discovery Service shall not support a
controller that exposes namespaces.
 dispersed namespace
A shared namespace that may be concurrently accessed by controllers in two or more NVM subsystems
(refer to section 8.1.9).
 dynamic controller
The controller is allocated on demand with no state (e.g., Feature settings) preserved from prior
associations.
 Domain
A domain is the smallest indivisible unit that shares state (e.g., power state, capacity information).
 embedded management controller
An embedded management controller is a Management Controller (refer to the NVM Express Management
Interface Specification) that provides an external management interface (e.g., Redfish ®), typically
implemented via commands to the Management Endpoint.
 emulated controller
An NVM Express controller that is defined in software. An emulated controller may or may not have an
underlying physical NVMe controller (e.g., physical PCIe function).
 Endurance Group
A portion of non-volatile storage in the NVM subsystem whose endurance is managed as a group. Refer to
section 3.2.3.
 Entry Key
A set of discovery information entry fields that allow for the unique identification of each discovery
information entry registered with the CDC or DDC. Refer to the Entry Key Type (EKTYPE) field.
 Exported Namespace
A namespace in an Exported NVM Subsystem.
 Exported NVM Resources
NVM resources created to enable remote access to physical NVM resources that includes:
a) Exported NVM Subsystems;
b) Exported Namespaces; and
c) Exported Ports.
