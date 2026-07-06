---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 661
headings: ["8.3.3.1.3 Clearing all Exported NVM Subsystems", "8.3.4 NVMe over Fabrics Secure Channel and In-band Authentication"]
---

# Source page 661

NVM Express® Base Specification, Revision 2.1

639
Figure 722: Example steps for revoking specific hosts access to an Exported NVM Subsystem
Administrator
Controller
8

An example flow for removing an Exported NVM Subsystem and associated Namespaces would be issuing:
1. a Manage Exported NVM Port command with a Delete operation (refer to section 5.3.9.1.2) for
each port associated with the Exported NVM Subsystem to be removed (refer to step ‘9’ of Figure
723).
2. a Manage Exported Namespace command with a Disassociate Namespace operation (refer to
section 5.3.7.1.2) for each Exported Namespaces associated with the Exported NVM Subsystem
to be removed (refer to step ‘10’ of Figure 723).
3. a Manage Exported NVM Subsystem command with a Delete operation (refer to section 5.3.8.1.1)
of a Manage Exported NVM Subsystem command (refer to step ‘11’ of Figure 723).
Figure 723: Example steps for removing an Exported NVM Subsystem
Administrator
Controller
10
9
11

8.3.3.1.3 Clearing all Exported NVM Subsystems
Issuing a Clear Exported NVM Resource Configuration command (refer to section 5.3.1) clears all Exported
NVM Subsystems (refer to Figure 724).
Figure 724: Example steps for clearing all Exported NVM Subsystems
Administrator
Controller

Upon successful completion of a Clear Exported NVM Resource Configuration command there are no
Exported NVM Subsystems.
 NVMe over Fabrics Secure Channel and In-band Authentication
NVMe over Fabrics supports both fabric secure channel (that includes authentication) and NVMe in -band
authentication. Fabric authentication is part of establishing a fabric secure channel via an NVMe Transport
