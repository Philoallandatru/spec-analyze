---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 61
headings: ["3.1.3.2 Administrative Controller"]
---

# Source page 61

NVM Express® Base Specification, Revision 2.1

39
 Administrative Controller
An Administrative controller is a controller whose intended purpose is to provide NVM subsystem
management capabilities. While an I/O controller may support these same management capabilities, an
Administrative controller has fewer mandatory capabilities. Unlike an I/O controller, an Administrative
controller does not support I/O commands that access to user data stored on an NVM subsystem’s non -
volatile storage medium. NVMe Transports may support a transport specific mechanism to allow an
Administrative controller to load a dedicated NVMe management driver instead of a generic NVMe driver
(refer to the applicable NVMe Transport binding specification for details).
Examples of management capabilities that may be supported by an Administrative controller include the
following.
• Ability to efficiently poll NVM subsystem health status via NVMe -MI using the NVMe -MI Send
command and the NVMe-MI Receive command  (refer to the NVM Subsystem Health Status Poll
section in the NVM Express Management Interface Specification);
• Ability to manage an NVMe enclosure via NVMe -MI using the NVMe -MI Send command and the
NVMe-MI Receive command;
• Ability to manage NVM subsystem namespaces using the Namespace Attachment command and
the Namespace Management command;
• Ability to perform virtualization management using the Virtualization Management command;
• Ability to reset an entire NVM subsystem using the NVM Subsystem Reset (NSSR) register  if
supported; and
• Ability to shutdown an entire NVM subsystem using the NVM Subsystem Shutdown (NSSD)
property, if supported.
An Administrative controller shall not support I/O Queues. Namespaces shall not be attached to an
Administrative controller.
An Administrative controller is required to support the mandatory Admin commands listed in  Figure 28. An
Administrative controller may support one or more I/O Command Sets. If an Administrative controller
supports an I/O Command Set, then only I/O Command Set specific Admin commands may be supported
since an Administrative controller only has an Admin Queue and no I/O Queues.
Figure 25 shows an NVM subsystem with one Administrative controller and two I/O controllers within an
NVM subsystem that contains a non-volatile storage medium and namespaces. I/O controller one has two
attached namespaces, private namespace A and shared namespace B. I/O controller two also has two
attached namespaces, private namespace C and shared namespace B.  An Administrative controller has
no attached namespaces. The Administrative controller in this example may be used for tasks such as NVM
subsystem namespace management and efficiently polling NVM subsystem health status via NVMe -MI.
While this example shows a single Administrative controller, an NVM subsystem may support zero or more
Administrative controllers.
