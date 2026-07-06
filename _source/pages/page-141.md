---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 141
headings: ["3.7.3 Queue Level Reset", "3.8 NVM Capacity Model", "3.8.1 Overview"]
---

# Source page 141

NVM Express® Base Specification, Revision 2.1

119
• set the CC.EN bit to ‘1’;
• wait for the CSTS.RDY bit to be set to ‘1’;
• configure the controller using Admin commands as needed;
• create I/O Completion Queues and I/O Submission Queues as needed; and
• proceed with normal I/O operations.
Note that all Controller Level Reset cases except a Controller Reset result in the controller immediately
losing communication with the host. In all these cases, the controller is unable to indicate any aborts or
update any completion queue entries.
If the host is no longer able to communicate with the controller before that host receives either :
• completions for all outstanding commands submitted to that controller (refer to section 3.4.5); or
• a CSTS.RDY bit value cleared to ‘0’,
then it is strongly recommended that the host take the steps described in section 9.6 to avoid possible data
corruption caused by interaction between outstanding commands and subsequent commands submitted
by that host to another controller.
 Queue Level Reset
The host may reset and/or reconfigure the I/O Submission and I/O Completion Queues by resetting them.
A queue level reset is performed by deleting and then recreating the queue. In this process, the host should
wait for all pending commands to the appropriate I/O Submission Queue(s) to complete.
To perform the queue level reset on a controller using the memory-based transport model, the host submits
the Delete I/O Submission Queue or Delete I/O Completion Queue command to the Admin Queue
specifying the identifier of the queue to be deleted. After successful command completion of the queue
delete operation, the host then recreates the queue by submitting the Create I/O Submission Queue or
Create I/O Completion Queue command. As part of the creation operation, the host may modify the
attributes of the queue. Note that if a queue level reset is performed on an I/O Completion Queue, the I/O
Submission Queues that are utilizing the I/O Completion Queue should be deleted before the I/O
Completion Queue is reset and recreated after the I/O Completion Queue is recreated. The behavior of an
I/O Submission Queue without a corresponding I/O Completion Queue is undefined.
To perform the queue level reset on a controller using the message-based transport model, the host sends
a Disconnect command to the I/O Queue which is to be deleted. After successful command completion  of
the Disconnect command, the host then recreates the I/O Submission Queue and I/O Completion Queue
by submitting the Connect command with a QID specified that is not 00h. As part of the Connect command,
the host may modify the attributes of the I/O queues.
The host should ensure that the appropriate I/O Submission Queue or I/O Completion Queue is idle before
deleting that queue. Submitting a queue deletion command causes any pending commands to be aborted
by the controller; this may or may not result in a completion queue entry being posted for the a borted
command(s).
3.8 NVM Capacity Model
 Overview
NVM subsystems may report capacity -related information for multiple entities within the NVM subsystem.
This capacity reporting model includes capacity reporting for the NVM subsystem, the domain (refer to
section 3.2.5), the Endurance Group (refer to section 3.2.3), the NVM Set (refer to section 3.2.2),
namespaces that contain formatted storage (refer to section 3.2.1), and the Media Unit (refer to section
1.5.54). Some, all, or none of this reporting may be supported by an NVM subsystem.
Figure 14 shows the hierarchical relationships of the entities within an NVM subsystem which are used to
manage NVM capacity.
