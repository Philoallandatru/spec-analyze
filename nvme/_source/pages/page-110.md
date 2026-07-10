---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 110
headings: ["3.3.2 Message-based Transport Queue Model (Fabrics)", "3.3.2.1 Capsules and Data Transfers"]
---

# Source page 110

NVM Express® Base Specification, Revision 2.1

88
Figure 74: Full Queue Definition

 Message-based Transport Queue Model (Fabrics)
For NVMe over Fabrics, a queue is a unidirectional communication channel that is used to send capsules
between a host and a controller. A host uses Submission Queues to send command capsules (refer to
section 3.3.2.1.1) to a controller. A controller uses Completion Queues to send response capsules (refer to
section 3.3.2.1.2) to a host. Submission and Completion Queues are created in pairs using the Connect
command (refer to section 3.3.2.2).
The NVMe Transport is responsible for delivering command capsules to the controller and notifying the
controller of capsule arrival in a transport-specific fashion.
Altering a command capsule between host submission to the Submission Queue and transport delivery of
that capsule to the controller results in undefined behavior.
NVMe Transports are not required to provide any additional end -to-end flow control. Specific NVMe
Transports may require low level flow control for congestion avoidance and reliability; any such additional
NVMe Transport flow control is outside the scope of this specification.
Flow control differs for Submission Queues (refer to section 3.3.2.1.1, section 3.3.2.6, and section 3.3.2.7)
and Completion Queues (refer to section 3.3.2.1.2, section 3.3.2.8, and section 3.3.1.2.1).
 Capsules and Data Transfers
This section describes capsules and data transfer mechanisms necessary to support message -based
transport queues. These mechanisms are used for Fabrics commands, Admin commands, and I/O
commands when using the message-based transport queue model.
A capsule is an NVMe unit of information exchanged between a host and a controller. A capsule may
contain commands, responses, SGLs, and/or data. The data may include user data (e.g., logical block data
and metadata that is transferred as a contiguous part of the logical block ) and data structures associated
with the command.
The capsule size for the Admin Queue commands and responses is fixed and defined in the NVMe
Transport binding specification. The controller indicates in the Identify Controller data structure the capsule
command and response sizes that the host shall use with I/O commands.
The controller shall support SGL based data transfers for commands on both the Admin Queue and I/O
Queues. Data may be transferred within the capsule or through memory transactions based on the
underlying NVMe Transport as indicated in the SGL descriptors associated with the command capsule. The
SGL types supported by an NVMe Transport are specified in the NVMe Transport binding specification.
The value of unused and not reserved capsule fields (e.g., the capsule is larger than the command /
response and associated data) is undefined and shall not be interpreted by the recipient.
occupiedoccupied
occupiedoccupied
Queue Base Address
Head (Consumer)
Tail (Producer)
emptyempty
occupiedoccupied
occupiedoccupied
……
occupiedoccupied
occupiedoccupied
occupiedoccupied
occupiedoccupied
occupiedoccupied
Queue Base Address
Head (Consumer)
Tail (Producer)
emptyempty
occupiedoccupied
occupiedoccupied
……
occupiedoccupied
occupiedoccupied
occupiedoccupied
