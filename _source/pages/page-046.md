---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 46
headings: ["2.2.5 Authentication", "2.3 NVM Storage Model", "2.3.1 Storage Entities"]
---

# Source page 46

NVM Express® Base Specification, Revision 2.1

24
Figure 10: Response Capsule Format

Completion Queue Entry Data (if present)
Byte 0 15 16 (N-1)
Response Capsule of Size N Bytes
NVMe Transports using the message-only transport model and message/memory transport model require
all SGLs sent from the host to the controller be transferred within the command. NVMe Transports may
optionally support the transfer of a portion or all data within the command and response capsules.
NVMe over Fabrics requires SGLs for all commands (Fabrics, Admin, and I/O). An SGL may specify the
placement of data within a capsule or the information required to transfer data using an NVMe Transport
specific data transfer mechanism (e.g., via memory tr ansfers as in RDMA). Each NVMe Transport binding
specification defines the SGLs used by a particular NVMe Transport and any capsule SGL and data
placement restrictions.
 Authentication
NVMe over Fabrics supports both fabric secure channel that includes authentication  (refer to section
8.3.4.1) and NVMe in -band authentication. An NVM subsystem may require a host to use fabric secure
channel, NVMe in -band authentication, or both. The Discovery Service indicates if fabric secure channel
shall be used for an NVM subsystem. The Connect response indicates if NVMe in-band authentication shall
be used with that controller.
A controller associated with an NVM subsystem that requires a fabric secure channel shall not accept any
commands (i.e., Fabrics commands, Admin commands, or I/O commands) on an NVMe Transport until a
secure channel is established. Following a Connect comm and, a controller that requires NVMe in -band
authentication shall not accept any commands on the queue created by that Connect command other than
authentication commands until NVMe in-band authentication has completed. Refer to section 8.3.4.
2.3 NVM Storage Model
 Storage Entities
The NVM storage model includes the following entities:
• NVM subsystems (refer to 1.5.66);
• Domains (refer to section 3.2.5);
• Endurance Groups (refer to section 3.2.3);
• Reclaim Groups and Reclaim Units (refer to section 3.2.4);
• NVM Sets (refer to section 3.2.2); and
• Namespaces (refer to section 3.2.1).
As illustrated below,
• each domain is contained in a single NVM subsystem;
• each Endurance Group is contained in a single domain and may contain either:
o one or more NVM Sets; or
o one or more Reclaim Groups;
• each NVM Set is contained in a single Endurance Group and each namespace is contained in a
single NVM Set. Each Media Unit is contained in a single Endurance Group; and
• each Reclaim Group is contained in a single Endurance Group, each Reclaim Unit is contained in
a Reclaim Group, and each namespace is contained in an Endurance Group within one or more
Reclaim Units of the Reclaim Groups in that Endurance Group.
