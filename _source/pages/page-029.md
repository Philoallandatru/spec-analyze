---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 29
headings: ["1.5.15 capsule", "1.5.16 Centralized Discovery controller (CDC)", "1.5.17 Channel", "1.5.18 command completion", "1.5.19 command submission", "1.5.20 controller", "1.5.21 Controller Reset", "1.5.22 Directive", "1.5.23 Direct Discovery controller (DDC)"]
---

# Source page 29

NVM Express® Base Specification, Revision 2.1

7
 capsule
An NVMe unit of information exchange used in NVMe over Fabrics. A capsule contains a command or
response and may optionally contain command/response data and SGLs.
 Centralized Discovery controller (CDC)
A Discovery controller that reports discovery information registered by Direct Discovery controllers and
hosts.
 Channel
A Channel represents a communication path between the controller and one or more Media Units in an
NVM subsystem.
 command completion
A command is completed when the controller has completed processing the command, has updated status
information in the completion queue entry, and has posted the completion queue entry to the associated
Completion Queue.
 command submission
For memory-based transport model (e.g. , PCIe) implementations, a command is submitted when a
Submission Queue Tail Doorbell write has completed that moves the Submission Queue Tail Pointer value
past the Submission Queue slot in which the command was placed.
For message-based transport model (e.g., NVMe over Fabrics) implementations, a command is submitted
when a host adds a capsule to a Submission Queue.
 controller
A controller is the interface between a host and an NVM subsystem. There are three types of controllers:
a) I/O controllers;
b) Discovery controllers; and
c) Administrative controllers.
A controller executes commands submitted by a host on a Submission Queue and posts a completion on
a Completion Queue. All controllers implement one Admin Submission Queue and one Admin Completion
Queue. Depending on the controller type, a controller may also implement one or more I/O Submission
Queues and I/O Completion Queues. When PCI Express is used as the transport, then a controller is a PCI
Express function.
 Controller Reset
Host modification of the CC property that clears CC.EN from ‘1’ to ‘0’ (refer to section 3.7.2).
 Directive
A method of information exchange between a host and either an NVM subsystem or a controller.
Information may be transmitted using the Directive Send and Directive Receive commands. A subset of I/O
commands may include a Directive Type field and a Directive Specific field to communicate more
information that is specific to the associated I/O command. Refer to section 8.1.8.
 Direct Discovery controller (DDC)
A Discovery controller that is capable of registering discovery information with a Centralized Discovery
controller.
