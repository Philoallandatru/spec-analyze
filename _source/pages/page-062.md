---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 62
headings: ["3.1.3.3 Discovery Controller"]
---

# Source page 62

NVM Express® Base Specification, Revision 2.1

40
Figure 25: NVM Subsystem with One Administrative and Two I/O Controllers

Figure 26 shows an NVM subsystem with one Administrative controller within an NVM subsystem that
contains no non-volatile storage medium or namespaces. The Administrative controller in this example may
be used to manage an NVMe enclosure using NVMe -MI. Since the Administrative controller is used for a
very specific dedicated purpose, the implementer of such an Administrative controller may choose to
implement only the mandatory capabilities along with the NVMe -MI Send and NVMe -MI Receive
commands.
Figure 26: NVM Subsystem with One Administrative Controller
Port
NVM Subsystem
Administrative
controller

 Discovery Controller
A Discovery controller is a controller used in NVMe over Fabrics . A Discovery controller enables a host to
discover other NVM subsystems or other Discovery subsystems. A Discovery controller  only implements
features related to Discovering other subsystems and does not implement I/O Queues , I/O commands, or
expose namespaces. The features supported by the Discovery controller are defined in section 3.1.3.6.
If the Discovery subsystem provides a unique  Discovery Service NQN (i.e., the NVM Subsystem NVMe
Qualified Name (SUBNQN) field in that Discovery subsystem’s Identify Controller data structure contains a
unique Discovery Service  NQN value), then that Discovery subsystem shall support both the unique
Discovery Service  NQN and the well -known Discovery Service NQN (i.e., nqn.2014 -
08.org.nvmexpress.discovery) being specified in the Connect command (refer to section 6.3) from the host.
If the Discovery subsystem does not provide a unique Discovery Service NQN (i.e., the SUBNQN field in
that Discovery subsystem’s Identify Controller data structure contains the well -known Discovery Service
NQN), then that Discovery subsystem shall support the well-known Discovery Service NQN being specified
in the Connect command from the host.
