---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 45
headings: ["2.2.3 Discovery Service", "2.2.4 Capsules and Data Transfer"]
---

# Source page 45

NVM Express® Base Specification, Revision 2.1

23
A controller is associated with exactly one host at a time. NVMe over Fabrics allows multiple hosts to
connect to different controllers in the NVM subsystem through the same NVM subsystem port. All other
aspects of NVMe over Fabrics multi-path I/O and namespace sharing (refer to section 2.4.1) are equivalent
to that of the memory-based transport model.
 Discovery Service
NVMe over Fabrics defines a discovery mechanism that a host uses to determine the NVM subsystems
that expose namespaces that the host may access. The Discovery Service provides a host with the
following capabilities:
• The ability to discover a list of NVM subsystems with namespaces that are accessible to the host;
• The ability to discover multiple paths to an NVM subsystem;
• The ability to discover controllers that are statically configured;
• The optional ability to establish explicit persistent connections to the Discovery controller; and
• The optional ability to receive Asynchronous Event Notifications from the Discovery controller.
A Discovery Service is an NVM subsystem that supports only Discovery controllers (refer to section 3.1.3.3),
and shall not support any other controller type.
The method that a host uses to obtain the information necessary to connect to the initial Discovery Service
is implementation specific. This information may be determined using a host configuration file, a hypervisor
or OS property, or some other mechanism.
 Capsules and Data Transfer
A capsule is an NVMe unit of information exchange used in NVMe over Fabrics. A capsule may be classified
as a command capsule or a response capsule. A command capsule contains a command (formatted as a
submission queue entry) and may optionally include SGLs or data. A response capsule contains a response
(formatted as a completion queue entry) and may optionally include data. Data refers to any data transferred
at an NVMe layer between a host and an NVM subsystem (e.g., logical block data or a data structur e
associated with a command). A capsule is independent of any underlying NVMe Transport unit (e.g.,
packet, message, or frame and associated headers and footers) and may consist of multiple such units.
Command capsules are transferred from a host to an NVM subsystem. The SQE contains an Admin
command, an I/O command, or a Fabrics command. The minimum size of a command capsule is NVMe
Transport binding specific, but shall be at least 64B in size. The maxi mum size of a command capsule is
NVMe Transport binding specific. The format of a command capsule is shown in Figure 9.
Figure 9: Command Capsule Format

Submission Queue Entry Data or SGLs (if present)
Byte 0 63 64 (N-1)
Command Capsule of Size N Bytes
Response capsules are transferred from an NVM subsystem to a host. The CQE is associated with a
previously issued Admin command, I/O command, or Fabrics command. The size of a response capsule is
NVMe Transport binding specific, but shall be at least 16B in size. The maximum size of a response capsule
is NVMe Transport binding specific. The format of a response capsule is shown in Figure 10.
