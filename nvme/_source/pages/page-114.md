---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 114
headings: ["3.3.2.2 Queue Creation"]
---

# Source page 114

NVM Express® Base Specification, Revision 2.1

92
The second example shows an 8K iB write where all of the data is transferred within the capsule. In this
case, the SGL descriptor is an SGL Data Block descriptor specifying an Offset of 20h based on an ICDOFF
value of 2h.
Figure 79: SGL Example Using In Capsule Data Transfer

Data Block A
Destination SGL Segment 0
Offset = 20h
SGL Data Block descriptor
specifies to transfer 8KiB within the
capsule at offset 20h.
Length = 8KiB
SGL Identifier = 01h
Submission Queue Entry
Byte 0
Data
63 64 (N-1)96
Undefined

 Queue Creation
Message-based controllers use the Connect command (refer to section 6.3) to create Admin Queues or I/O
Queues. The creation of an Admin Queue establishes an association between a host and the corresponding
controller. The message -based transport queue model does not support the Admin Submission Queue
Base Address (ASQ), Admin Completion Queue Base Address (ACQ), and Admin Queue Attributes (AQA)
properties as all information necessary to establish an Admin Queue is contained in the Connect command.
The message-based transport queue model does not support the Admin commands associated with I/O
Queue creation and deletion (i.e., Create I/O Completion Queue, Create I/O Submission Queue, Delete I/O
Completion Queue, Delete I/O Submission Queue).
An NVMe Transport connection is established between a host and an NVM subsystem prior to the transfer
of any capsules or data. The mechanism used to establish an NVMe Transport connection is NVMe
Transport specific and defined by the corresponding NVMe Tra nsport binding specification. The NVMe
Transport may require a separate NVMe Transport connection for each Admin Queue or I/O Queue or may
utilize the same NVMe Transport connection for all Admin and I/O Queues associated with a particular
controller. An NVMe Transport may also require that NVMe layer information be passed between the host
and controller in the process of establishing an NVMe Transport connection (e.g., exchange queue size to
appropriately size send and receive buffers).
The Connect command specifies the Queue ID and type (Admin or I/O), the size of the Submission and
Completion Queues, queue attributes, Host NQN, NVM Subsystem NQN, and Host Identifier. The Connect
command may specify a particular controller if the NVM subsystem supports a static controller model. The
Connect response indicates whether the connection was successfully established as well as whether NVMe
in-band authentication is required.
The Connect command is submitted to the same Admin Queue or I/O Queue that the Connect command
creates. The underlying NVMe Transport connection that is used for that queue is created first and the
Connect command and response capsules are sent over that N VMe Transport connection. The Connect
command shall be sent once to a queue.
When a Connect command successfully completes, the corresponding Submission and Completion
Queues are created. If NVMe in-band authentication is required as indicated in the Connect response, then
