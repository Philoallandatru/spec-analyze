---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 42
headings: []
---

# Source page 42

NVM Express® Base Specification, Revision 2.1

20
Figure 6: Queue Pair Example, 1:1 Mapping


Figure 7: Queue Pair Example, n:1 Mapping

A Submission Queue (SQ) is a circular buffer with a fixed slot size that the host software uses to submit
commands for execution by the controller. The host software updates the appropriate SQ Tail doorbell
register when there are one to n new commands to execute. The previous SQ Tail value is overwritten in
the controller when there is a new doorbell register write. The controller fetches SQ entries in order from
the Submission Queue and may execute those commands in any order.
Each submission queue entry is a command. Commands are 64 bytes in size. The physical memory
locations in memory to use for data transfers are specified using Physical Region Page (PRP) entries or
Scatter Gather Lists  (SGL). Each command may include two PRP entries or one Scatter Gather List
segment. If more than two PRP entries are necessary to describe the data buffer, then a pointer to a PRP
List that describes a list of PRP entries is provided. If more than one SGL segment is necessary to describe
the data buffer, then the SGL segment provides a pointer to the next SGL segment.
A Completion Queue (CQ) is a circular buffer with a fixed slot size used to post status for completed
commands. A completed command is uniquely identified by a combination of the associated SQ identifier
and command identifier that is assigned by host soft ware. In the memory-based transport model multiple
Submission Queues may be associated with a single Completion Queue. A configuration with a single
Completion Queue may be used where a single worker thread processes all command completions via one
Completion Queue even when those commands originated from multiple Submission Queues. The CQ
Head pointer is updated by host software after processing completion queue entries indicating the last free
Host
Controller
Controller Mgmt Core 0 Core 1 Core N-1
Admin
Submission
Queue
Admin
Completion
Queue
I/O
Submission
Queue 1
I/O
Completion
Queue 1
I/O
Submission
Queue 2
I/O
Completion
Queue 2
I/O
Submission
Queue N
I/O
Completion
Queue N
Host
Controller
Core BCore A
I/O
Submission
Queue X
I/O
Completion
Queue W
I/O
Submission
Queue M
I/O
Completion
Queue N
I/O
Submission
Queue Y
I/O
Submission
Queue Z
Controller Mgmt
Admin
Submission
Queue
Admin
Completion
Queue
