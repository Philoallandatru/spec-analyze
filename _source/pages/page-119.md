---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 119
headings: ["3.3.2.9 Transport Requirements", "3.3.3 Queueing Attributes", "3.3.3.1 Queue Size", "3.3.3.2 Queue Identifier", "3.3.3.3 Queue Priority"]
---

# Source page 119

NVM Express® Base Specification, Revision 2.1

97
The value of the Maximum Outstanding Commands (MAXCMD) field in the Identify Controller data structure
indicates the maximum number of commands that the controller processes at one time for a particular I/O
Queue. The host may use this value to size I/O Completion Queues and optimize the number of commands
submitted at one time per queue to achieve the best performance.
Altering a response capsule between controller submission to the Completion Queue and transport delivery
of that capsule to the host results in undefined behavior.
 Transport Requirements
This section defines requirements that all NVMe Transports that support an NVMe over Fabrics
implementation shall meet.
The NVMe Transport may support NVMe Transport error detection and report errors to the NVMe layer in
command status values. The controller may record NVMe Transport specific errors in the Error Information
log page (refer to section 5.1.12.1.2). Transport errors that cause loss of a message or loss of data in a
way that the low-level NVMe Transport cannot replay or recover should cause:
• the deletion of the individual I/O Queues (refer to section  3.3.2.4) and the associated NVMe
Transport connection on which that NVMe Transport level error occurred; or
• termination of the NVMe Transport connection and the association between the host and controller.
The NVMe Transport shall provide reliable delivery of capsules between a host and NVM subsystem (and
allocated controller) over each connection. The NVMe Transport may deliver command capsules in any
order on each queue except for I/O commands that are part of fused operations (refer to section 3.4.2).
For command capsules that are part of fused operations for I/O commands, the NVMe Transport:
1. shall deliver the first and second command capsules for each fused operation to the queue in -
order; and
2. shall not deliver any other command capsule for the same Submission Queue between delivery of
the two command capsules for a fused operation.
The NVMe Transport shall provide reliable delivery of response capsules from an NVMe subsystem to a
host over each connection. The NVMe Transport shall deliver response capsules that include an SQ Head
Pointer (SQHD) value to the host in-order; this includes all Connect response capsules and all Disconnect
response capsules.
 Queueing Attributes
 Queue Size
The Queue Size is indicated in a 16-bit 0’s based field that indicates the number of slots in the queue. The
minimum size for a queue is two slots. The maximum size for either an I/O Submission Queue or an I/O
Completion Queue is defined as 65,536 slots, limited by the maximum queue size supported by the
controller that is reported in the CAP.MQES field. The maximum size for the Admin Submission Queue and
Admin Completion Queue is defined as 4,096 slots. One slot in each queue is not available for use due t o
Head and Tail entry pointer definition.
For Message-based controllers, the maximum size for the Admin Submission Queue is limited by the value
indicated in the ASQSZ field in the Discovery Log Page Entry data structure (refer to Figure 294).
 Queue Identifier
Each queue is identified through a 16-bit ID value that is assigned to the queue when it is created. Both I/O
Submission Queue identifiers and I/O Completion Queue identifiers are a value from 1 to 65,535.
 Queue Priority
If the weighted round robin with urgent priority class arbitration mechanism is supported, then host software
may assign a queue priority service class of Urgent, High, Medium, or Low. If the weighted round robin with
