---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 44
headings: ["2.2.2 NVM Subsystem Ports for Fabrics"]
---

# Source page 44

NVM Express® Base Specification, Revision 2.1

22
Figure 8: NVMe over Fabrics Layering
NVMe Architecture, Queuing Interface
Admin Command & I/O Command Sets,
Properties
Fabric Specific Properties,
Transport Specific Features/Specialization
NVMe Transport Binding Services
NVMe Transport
NVMe over Fabrics
(Message-based
Model)
NVMe Transport
Specification
NVMe Transport
Fabric Protocol
(may include multiple fabric protocol layers)
Fabric
Fabric Physical
(e.g., Ethernet, InfiniBand, Fibre Channel)


 NVM Subsystem Ports for Fabrics
An NVM subsystem presents a collection of one to (64K i - 16) controllers which are used to access
namespaces. The controllers may be associated with hosts through one to 64K i NVM subsystem ports.
An NVM subsystem port is a protocol interface between an NVM subsystem and a fabric. An NVM
subsystem port is a collection of one or more physical fabric interfaces that together act as a single protocol
interface. When link aggregation (e.g., Ethernet) is  used, the physical ports for the group of aggregated
links constitute a single NVM subsystem port.
An NVM subsystem contains one or more NVM subsystem ports.
Each NVM subsystem port has a 16-bit port identifier (Port ID). An NVM subsystem port is identified by the
NVM Subsystem NVMe Qualified Name ( NQN) and Port ID. The NVM subsystem ports of an NVM
subsystem may support different NVMe Transports. An NVM subsystem port may support multiple NVMe
Transports if more than one NVMe Transport binding specifications exist for the underlying fabric (e.g., an
NVM subsystem port identified by a Port ID may support both iWARP and RoCE). An NVM subsystem
implementation may bind specific controllers to specific NVM subsystem ports or allow the flexible allocation
of controllers between NVM subsystem ports, however, once connected, each specific controller is bound
to a single NVM subsystem port.
