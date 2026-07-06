---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 117
headings: ["3.3.2.5 Submission Queue Flow Control Negotiation", "3.3.2.6 Submission Queue Flow Control"]
---

# Source page 117

NVM Express® Base Specification, Revision 2.1

95
The response to the Disconnect command is the last I/O Completion Queue entry processed by the host
for an I/O Queue. To avoid command aborts, the host should wait for all outstanding commands on an I/O
Queue to complete before sending the Disconnect command.
If the controller terminates an NVMe Transport connection or detects an NVMe Transport connection loss,
then the controller shall stop processing all commands received on I/O Queues associated with that NVMe
Transport connection within the time reported in the CQT field (refer to Figure 312), if non-zero.
If the host terminates an NVMe Transport connection or detects an NVMe Transport connection loss before
the responses are received for all outstanding commands submitted to the associated I/O Queue  (refer to
section 3.4.5), then it is strongly recommended that the host take the steps described in section 9.6 to avoid
possible data corruption caused by interaction between outstanding commands and subsequent
commands submitted by that host to another controller.
 Submission Queue Flow Control Negotiation
Use of Submission Queue (SQ) flow control is negotiated for each queue pair by the Connect command
and the controller response to the Connect command. SQ flow control shall be used unless it is disabled
as a result of that negotiation. If SQ flow control i s disabled, then the Submission Queue Head Pointer
(SQHD) field is reserved in all Fabrics response capsules for that queue pair after the response to the
Connect command (i.e., in all subsequent response capsules for that queue pair, the controller shall clear
the SQHD field to 0h and the host should ignore the SQHD field).
If the host requests that SQ flow control be disabled for a queue pair, then the host should size each
Submission Queue to support the maximum number of commands that the host could have outstanding at
one time for that Submission Queue.
The maximum size of the Admin Submission Queue is specified in the Admin Max SQ Size (ASQSZ) field
of the Discovery Log Page Entry for the NVM subsystem (refer to section 5.1.12.3.1).
The maximum size of an I/O Submission Queue is specified in the Maximum Queue Entries Supported
(MQES) field of the Controller Capabilities (CAP) property for the controller (refer to section  3.1.4.1).
The value of the Maximum Outstanding Commands (MAXCMD) field in the Identify Controller data structure
indicates the maximum number of commands that the controller processes at one time for a particular I/O
Queue. The host may use this value to size I/O Submission Queues and optimize the number of commands
submitted at one time per queue to achieve the best performance.
If SQ flow control is disabled, then the host should limit the number of outstanding commands for a queue
pair to be less than the size of the Submission Queue. If the controller detects that the number of
outstanding commands for a queue pair is greater than or equal to the size of the Submission Queue, then
the controller shall:
a) stop processing commands and set the Controller Fatal Status (CSTS.CFS) bit to ‘1’ (refer to
section 9.5); and
b) terminate the NVMe Transport connection and end the association between the host and the
controller.
 Submission Queue Flow Control
This section applies only to Submission Queues that use SQ flow control.
The Submission Queue has a Head entry pointer and a Tail entry pointer that are used to manage the
queue and determine the number of Submission Queue capsules available to the host for new submissions.
The Head and Tail entry pointers are initialized to 0h when a queue is created. All arithmetic operations
and comparisons on entry pointers are performed modulo the queue size with queue wrap conditions taken
into account. The host increments the Tail entry pointer when the host adds a capsule to a queue. The
controller increments the Head entry pointer when that controller removes a capsule from the queue.
The Submission Queue Head entry pointer is maintained by the controller and is communicated to the host
in the SQHD field of completion queue entries. The host uses the received SQHD values for Submission
Queue management (e.g., to determine whether the Submission Queue is full).
