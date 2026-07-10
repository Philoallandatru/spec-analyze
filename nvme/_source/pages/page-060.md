---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 60
headings: ["3.1.3.1 I/O Controller"]
---

# Source page 60

NVM Express® Base Specification, Revision 2.1

38
arbitrarily by the controller . Data associated with the processing of a command may or may not be
committed to the NVM subsystem non-volatile storage medium in the order that commands are submitted.
Host software submits commands of higher priorities to the appropriate Submission Queues. Priority is
associated with the Submission Queue itself, thus the priority of the command is based on the Submission
Queue to which that command was submitted. The co ntroller arbitrates across the Submission Queues
based on fairness and priority according to the arbitration scheme specified in section 3.4.4.
Upon completion of the command execution by the NVM subsystem, the controller presents completion
queue entries to the host through the appropriate Completion Queues. Transport specific methods (e.g.,
PCIe interrupts) are used to notify the host of completion queue entries to process (refer to the appropriate
Transport specification).
There are no ordering restrictions for completions to the host. Each completion queue entry identifies the
Submission Queue Identifier and Command Identifier of the associated command. Host software uses this
information to correlate the completions with the commands submitted to the Submission Queue(s).
Host software is responsible for creating I/O Submission Queues and I/O Completion Queues prior to using
those queue pairs to submit  commands to the controller. I/O Submission Queues and I/O Completion
Queues are created using the Create I/O Submission Queue command (refer to section 5.2.2) and the
Create I/O Completion Queue command (refer to section 5.2.1).
 I/O Controller
An I/O controller is a controller that supports commands that provide access to user data stored on an NVM
subsystem’s non -volatile storage medium  using an I/O command set  and may support commands that
provide management capabilities.
An I/O controller may simultaneously support multiple I/O Command Sets. The I/O Command Sets that the
controller supports and which of these I/O Command Sets the controller simultaneously supports is reported
in the Identify I/O Command Set data structure (refer to section 5.1.13.2.19). The contents of the Identify
I/O Command Set data structure are not required to be the same for all controllers in an NVM subsystem.
Figure 24 shows an NVM subsystem with three I/O controllers. I/O controller one has two attached
namespaces, private namespace A and shared namespace B. I/O controller two also has two attached
namespaces, private namespace C and shared namespace B. I/O controller three has no attached
namespaces. At some later point in time shared namespace B may be attached to I/O controller three.
Figure 24: NVM Subsystem with Three I/O Controllers
