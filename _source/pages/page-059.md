---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 59
headings: ["3.1.3 Controller Types"]
---

# Source page 59

NVM Express® Base Specification, Revision 2.1

37
An association between a host and controller is terminated if:
• the controller is shutdown as described in section 3.6.2;
• a Controller Level Reset occurs;
• the NVMe Transport connection is lost between the host and controller for the Admin Queue; or
• an NVMe Transport connection is lost between the host and controller for any I/O Queue and the
host or controller does not support individual I/O Queue deletion (refer to section 3.3.2.4).
There is no explicit NVMe command that breaks the NVMe Transport association between a host and
controller. The Disconnect command (refer to section 6.4) provides a method to delete an I/O Queue (refer
to section 3.3.2.4). While a controller is associated with a host, that controller is busy, and no other
associations may be made with that controller.
To use the dynamic controller model, the host specif ies a controller identifier of FFFFh when using the
Fabrics Connect command (refer to section 6.3) to establish an association with an NVM subsystem.
When using the static controller model with a fabric connected controller, the state that persists across
associations is any state that persists across a Controller Level Reset. Additionally, different controllers
may present different Feature settings or namespace attachments to the same host. The NV M subsystem
may allocate particular controllers to specific hosts.
While allocation of static controllers to hosts are expected to be durable (so that hosts can expect to form
associations to the same controllers repeatedly (e.g., after each host reboot)), the NVM subsystem may
remove the host allocation of a controller that is not in use at any time for implementation specific reasons
(e.g., controller resource reclamation, subsystem reconfiguration).
 Controller Types
As shown in  Figure 23Figure 23, there are three types of controllers. An I/O controller  (refer to section
3.1.3.1) is a controller that supports commands that provide access to user data stored on an NVM
subsystem’s non -volatile storage medium and may support commands that provide management
capabilities. An Administrative controller (refer to section 3.1.3.2) is a controller that support s commands
that provide management capabilities, but does not support I/O commands that access to user data stored
on an NVM subsystem’s non-volatile storage medium. A Discovery controller (refer to section 3.1.3.3) is a
controller used in NVMe over Fabrics to provide access to a Discovery log page.
Figure 23: Controller Types

The Controller Type (CNTRLTYPE) field in the Identify Controller data structure indicates a controller’s
type. Regardless of controller type, all controllers implement one Admin Submission Queue and one Admin
Completion Queue. Depending on the controller t ype, a controller may also support one or more I/O
Submission Queues and I/O Completion Queues.
When using a memory-based transport implementation (e.g., PCIe), host software submits commands to a
controller through pre-allocated Submission Queues. A controller is alerted to newly submitted commands
through SQ Tail Doorbell register writes. The difference between the previous doorbell register value and
the current register write indicates the number of commands that were submitted.
A controller fetches commands from the Submission Queue(s) and processes them. Except for fused
operations, there are no ordering restrictions for processing of commands within or across Submission
Queues. Host software should not submit commands to a Subm ission Queue that may not be re -ordered
