---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 118
headings: ["3.3.2.7 Submission Queue Head Pointer Update Optimization", "3.3.2.8 Completion Queue Considerations"]
---

# Source page 118

NVM Express® Base Specification, Revision 2.1

96
The Submission Queue Tail entry pointer is local to the host and is not communicated to the controller.
The Submission Queue is full when the Head entry pointer equals one more than the Tail entry pointer (i.e.,
incrementing the Tail entry pointer has caused it to wrap around to just behind the Head entry pointer). A
full Submission Queue contains one less c apsule than the queue size. A host may continue to submit
commands to a Submission Queue as long as the queue is not full.
If the controller detects that the host has submitted a command capsule to a full Submission Queue, then
the controller shall:
a) stop processing commands and set the Controller Fatal Status (CSTS.CFS) bit to ‘1’ (refer to
section 9.5); and
b) terminate the NVMe Transport connection and end the association between the host and the
controller.
The Submission Queue is empty when the Head entry pointer equals the Tail entry pointer.
 Submission Queue Head Pointer Update Optimization
Submission Queue Head Pointer update optimization does not apply to queue pairs for which Submission
Queue (SQ) flow control is disabled, as the SQHD field is reserved if SQ flow control is disabled, refer to
section 3.3.2.5 and to section 6.3.
The NVMe Transport may omit transmission of the SQHD value for a response capsule that:
a) contains a Generic Command status (i.e., Status Code Type 0h) indicating successful completion
of a command (i.e., Status Code 00h);
b) is not a Connect response capsule; and
c) is not a Disconnect response capsule.
If a new SQHD value is not received in a response capsule, the host continues to use its previous SQHD
value. Thus, at the NVMe layer there is a logical progression of SQHD values despite the fact that the
NVMe Transport may not actually transfer the SQHD value in each response capsule.
The NVMe Transport may deliver response capsules that do not contain an SQHD value to the host in any
order. The applicable NVMe Transport binding specification defines how presence versus absence of an
SQHD value in a response capsule is indicated by the NVMe Transport.
Periodic SQHD updates at the host are required to avoid Submission Queue (SQ) starvation as SQHD
value transmission in responses is the only means of releasing SQ slots for host reuse.
An NVMe Transport may transmit an SQHD value in every response capsule. If an NVMe Transport does
not transmit an SQHD value in every response capsule, then an SQHD value should be transmitted
periodically (e.g., in at least one of every n response capsule s on a CQ, where n is 10% of the size of the
associated SQ) or more often. An SQHD value should always be transmitted if 90% or more of the slots in
the associated SQ are occupied at the subsystem.
 Completion Queue Considerations
Completion Queue flow control  (refer to section 3.3.1.2.1) is not used in the message -based transport
queue model. Message-based transport Completion Queues do not use either Head entry pointers or Tail
entry pointers.
The host should size each Completion Queue to support the maximum number of commands that the host
could have outstanding at one time for a particular Submission Queue. The Completion Queue size may
be larger than the size of the corresponding Submission Queue to accommodate responses for commands
that are being processed by the controller in addition to responses for commands that are still in the
Submission Queue.
If the size of a Completion Queue is too small for the number of outstanding commands and the controller
submits a response capsule to a full Completion Queue, then the results are undefined.
