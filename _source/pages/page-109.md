---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 109
headings: ["3.3.1.3 Queue Abort", "3.3.1.4 Empty Queue", "3.3.1.5 Full Queue"]
---

# Source page 109

NVM Express® Base Specification, Revision 2.1

87
 Queue Abort
To abort a large number of commands, the host may use:
• the Cancel command (refer to section 7.1); or
• delete and recreate the I/O Submission Queue (refer to section 3.7.3).
Specifically, to abort all commands that are submitted to an I/O Submission Queue, host software should:
• issue a Cancel command to that queue with the Cancel Action set to Multiple Command Cancel
and the NSID field set to FFFFFFFFh; or
• issue a Delete I/O Submission Queue command for that queue. After that submission queue has
been successfully deleted, indicating that all commands have been completed or aborted, then
host software should recreate the queue by submitting a Create I/O Submission Queue command.
Host software may then re-submit commands to the associated I/O Submission Queue.
If the host is no longer able to communicate with the controller before that host receives either:
• completions for all outstanding commands submitted on that I/O Submission Queue (refer to
section 3.4.5); or
• a successful completion for the Delete I/O Submission Queue command for that I/O Submission
Queue,
then it is strongly recommended that the host take the steps described in section 9.6 to avoid possible data
corruption caused by interaction between outstanding commands and subsequent commands submitted
by that host to another controller.
 Empty Queue
The queue is Empty when the Head entry pointer equals the Tail entry pointer. Figure 73 defines the Empty
Queue condition.
Figure 73: Empty Queue Definition

 Full Queue
The queue is Full when the Head equals one more than the Tail. The number of entries in a queue when
full is one less than the queue size. Figure 74 defines the Full Queue condition.
Note: Queue wrap conditions shall be taken into account when determining whether a queue is Full.
emptyempty
emptyempty
Queue Base Address
Head (Consumer) Tail (Producer)
emptyempty
emptyempty
emptyempty
……
emptyempty
emptyempty
emptyempty
emptyempty
emptyempty
Queue Base Address
Head (Consumer) Tail (Producer)
emptyempty
emptyempty
emptyempty
……
emptyempty
emptyempty
emptyempty
