---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 115
headings: ["3.3.2.3 Queue Initialization and Queue State", "3.3.2.4 I/O Queue Deletion"]
---

# Source page 115

NVM Express® Base Specification, Revision 2.1

93
NVMe in-band authentication shall be performed before the queues may be used to perform other Fabrics
commands, Admin commands, or I/O commands.
Once a Connect command for an Admin Queue has completed successfully (and NVMe in -band
authentication, if required, has succeeded), only Fabrics commands may be submitted until the controller
is ready (CSTS.RDY = 1). Both Fabrics commands and Admin commands may be submitted to the Admin
Queue while the controller is ready. A Connect command for an I/O Queue may be subm itted after the
controller is ready. Once a Connect command for an I/O Queue has completed successfully (and NVMe in-
band authentication, if required, has succeeded), I/O commands may be submitted to the queue.
The Connect response contains the controller ID allocated to the host.
After an Admin Queue is created on a controller, all subsequent Connect commands sent from the same
host to that controller, to create an I/O Queue, are required to:
• utilize the same NVMe Transport;
• have the same Host NQN;
• have the same NVM Subsystem NQN; and
• either have the:
o same Host Identifier value; or
o a Host Identifier value of 0h, if supported (refer to section 5.1.12.3.1).
 Queue Initialization and Queue State
When a Connect command successfully completes, the corresponding Admin Submission and Completion
Queue or I/O Submission and Completion Queues are created. If the host sends a Connect command
specifying the Queue ID of a queue which already exists, then  the controller shall abort the command with
a status code of Command Sequence Error.
The Authentication Requirements (AUTHREQ) field in the Connect response indicates if NVMe in -band
authentication is required. If AUTHREQ is cleared to 0h, the created queue is ready for use after the
Connect command completes successfully. If AUTHREQ is set to a non -zero value, the created queue is
ready for use after NVMe in-band authentication has been performed successfully using the Authentication
Send and Authentication Receive Fabrics commands.
If a controller requires or is undergoing NVMe in -band authentication for a queue pair, then a controller
shall abort all commands received on that queue other than authentication commands with a status code
of Authentication Required. After the NVMe in -band authentication has been performed successfully on a
queue, then a controller shall abort all authentication commands on that queue with a status code of
Command Sequence Error.
When an Admin Queue is first created, the associated controller is disabled (i.e., CC.EN is initialized to ‘0’).
A disabled controller shall abort all commands other than Fabrics commands on the Admin Queue with a
status code of Command Sequence Error. After the controller is enabled, it shall accept all supported Admin
commands in addition to Fabrics commands.
A created I/O queue shall abort all commands  with a status code of Command Sequence Error if the
associated controller is disabled.
 I/O Queue Deletion
NVMe over Fabrics deletes an individual I/O Queue and may delete the associated NVMe Transport
connection as a result of:
• the exchange of a Disconnect command and response (refer to section 6.4) between a host and
controller; or
• the detection and processing of a transport error on an NVMe Transport connection.
The host indicates support for the deletion of an individual I/O Queue by setting the Individual I/O Queue
Deletion Support (INDIVIOQDELS) bit to ‘1’ in the CATTR field in the Connect command (refer to Figure
545) used to create the Admin Queue. The controller indicates support for the deletion of an individual I/O
