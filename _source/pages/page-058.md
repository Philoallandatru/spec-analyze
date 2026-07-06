---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 58
headings: ["3 NVM Express Architecture", "3.1 NVM Controller Architecture", "3.1.1 Memory-Based Controller Architecture (PCIe)", "3.1.2 Message-Based Controller Architecture (Fabrics)"]
---

# Source page 58

NVM Express® Base Specification, Revision 2.1

36
3 NVM Express Architecture
3.1 NVM Controller Architecture
A controller is the interface between a host and an NVM subsystem. This specification defines two controller
models, the static controller model and the dynamic controller model. All controllers in an NVM subsystem
shall support the same controller model.
In an NVM subsystem that supports the static controller model, state (e.g., controller ID, saved Feature
settings) is preserved:
• across a Controller Level Reset for memory-based controllers and message-based controllers; and
• from prior associations for message-based controllers.
In an NVM subsystem that supports the dynamic controller model, the NVM subsystem allocates controllers
on demand with no state preserved from prior associations.
 Memory-Based Controller Architecture (PCIe)
Memory-based controllers shall support only the static controller model.
 Message-Based Controller Architecture (Fabrics)
Message-based controllers, other than Discovery controllers, may support either the dynamic controller
model or the static controller model. A Discovery controller shall support only the dynamic controller model.
In an NVM subsystem that supports the static controller model, each controller that is allocated to a
particular host may have different state at the time the association is established (e.g., each controller may
have state that is preserved from a prior a ssociation). The controllers within such an NVM subsystem are
distinguished by their controller identifier. The host may request a particular controller based on the
Controller ID (refer to the CNTLID field in Figure 546).
In an NVM subsystem that supports the dynamic controller model, each controller is allocated by the NVM
subsystem on demand  with no state (e.g., Controller ID, Feature settings) preserved from prior
associations. In this model, all controllers allocated to a specific host have the same state at the time the
association is established, including Feature settings. The initial set of attached namespaces should be the
same for all controllers that are allocated to a specific host and accessed via the same NVM s ubsystem
port. The initial set of attached namespaces may differ among controllers that are each accessed via a
different NVM subsystem port. Changes to a dynamic controller  (e.g., attached namespaces, Feature
settings) after th e association is established do not impact other dynamic controllers  in that NVM
subsystem.
An association is established between a host and a controller when the host connects to a controller’s
Admin Queue using the Fabrics Connect command (refer to section 6.3). Within the Connect command,
the host specifies the Host NQN, NVM Subsystem NQN, Host Identifier, and may request a specific
Controller ID (e.g., the static controller model is being used) or may request a connection to any available
controller (e.g., the dynamic controller model is being used). A controller has only one association at a time.
While an association exists between a host and a controller, only that host may establish connections with
I/O Queues of that controller.  To establish a new connection with I/O Queues of that controller, the host
sends subsequent Connect commands using the same NVM subsystem port, NVMe Transport type, and
NVMe Transport address and specifies the:
• same Host NQN;
• same NVM Subsystem NQN;
• same Controller ID; and
• either the:
o same Host Identifier; or
o a Host Identifier value of 0h, if supported (refer to section 5.1.12.3.1).
