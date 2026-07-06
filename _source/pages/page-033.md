---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 33
headings: ["1.5.60 MMHD", "1.5.61 MMHS", "1.5.62 namespace", "1.5.63 namespace ID (NSID)", "1.5.64 NVM", "1.5.65 NVM Set", "1.5.66 NVM subsystem", "1.5.67 NVM subsystem port", "1.5.68 NVMe over Fabrics", "1.5.69 NVMe Transport", "1.5.70 NVMe Transport binding specification", "1.5.71 participating NVM subsystem"]
---

# Source page 33

NVM Express® Base Specification, Revision 2.1

11
 MMHD
A Migration Management Host associated with a Migration Management Controller in a Destination NVM
Subsystem. Refer to section 8.1.12.
 MMHS
A Migration Management Host associated with a Migration Management Controller in a Source NVM
Subsystem. Refer to section 8.1.12.
 namespace
A set of resources that may be directly accessed by a host (e.g., formatted non-volatile storage).
 namespace ID (NSID)
An identifier used by a controller to provide access to a namespace or the name of the field in the SQE that
contains the namespace identifier (refer to Figure 92). Refer to section 3.2.1 for the definitions of valid
NSID, invalid NSID, active NSID, inactive NSID, allocated NSID, and unallocated NSID.
 NVM
NVM is an acronym for non-volatile memory.
 NVM Set
A portion of NVM from an Endurance Group. Refer to section 3.2.2.
 NVM subsystem
An NVM subsystem includes one or more domains, one or more controllers, zero or more namespaces,
and one or more ports . An NVM subsystem may include a non-volatile storage medium and an interface
between the controller(s) in the NVM subsystem and non-volatile storage medium.
 NVM subsystem port
An NVMe over Fabrics protocol interface between an NVM subsystem and a fabric. An NVM subsystem
port is a collection of one or more physical fabric interfaces that together act as a single interface.
 NVMe over Fabrics
An implemen tation of the NVM Express interface that complies with either the message -only transport
model or the message/memory transport model (refer to Figure 4 and section 2.2).
 NVMe Transport
A protocol layer that provides reliable delivery of data, commands, and responses between a host and an
NVM subsystem. The NVMe Transport layer is layered on top of the fabric. It is independent of the fabric
physical interconnect and low-level fabric protocol layers.
 NVMe Transport binding specification
A specification of reliable delivery of data, commands, and responses between a host and an NVM
subsystem for an NVMe Transport. The binding may exclude or restrict functionality based on the NVMe
Transport’s capabilities.
 participating NVM subsystem
An NVM subsystem that participates in (i.e., contains controllers that provide access to) a dispersed
namespace.
