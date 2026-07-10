---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 698
headings: ["Annex B. Host Considerations (Informative)", "B.1 Basic Steps when Building a Command", "B.2 Creating an I/O Submission Queue"]
---

# Source page 698

NVM Express® Base Specification, Revision 2.1

676
Annex B. Host Considerations (Informative)
B.1 Basic Steps when Building a Command
When host software builds a command for the controller to execute, it first checks to make sure that the
appropriate Submission Queue (SQ) is not full. The Submission Queue is full when the number of entries
in the queue is one less than the queue size. Once an empty slot (pFreeSlot) is available:
1. Host software builds a command at SQ[pFreeSlot] with:
a. CDW0.OPC is set to the appropriate command to be executed by the controller;
b. CDW0.FUSE is set to the appropriate value, depending on whether the command is a
fused operation;
c. CDW0.CID is set to a unique identifier for the command when combined with the
Submission Queue identifier;
d. The Namespace Identifier, NSID field, is set to the namespace the command applies to;
e. MPTR shall be filled in with the offset to the beginning of the Metadata Region, if there is
a data transfer and the namespace format contains metadata as a separate buffer;
f. PRP1 and/or PRP2 (or SGL Entry 1 if SGLs are used) are set to the source/destination of
data transfer, if there is a data transfer; and
g. CDW10 – CDW15 are set to any command specific information;
2. Host software then completes a transport specific action in order to submit the command for
processing.
B.2 Creating an I/O Submission Queue
This example describes how host software creates an I/O Submission Queue that utilizes non -contiguous
PRP entries. Creating an I/O Submission Queue that utilizes a PRP List is only valid if the controller
supports non-contiguous queues as indicated in CAP.CQR.
Prior to creating an I/O Submission Queue, host software shall create the I/O Completion Queue that the
SQ uses with the Create I/O Completion Queue command.
To create an I/O Submission Queue, host software builds a Create I/O Submission Queue command for
the Admin Submission Queue. Host software builds the Create I/O Submission Queue command in the
next free Admin Submission Queue command location. The attributes of the command are:
• CDW0.OPC is set to 01h;
• CDW0.FUSE is cleared to 00b indicating that this is not a fused operation;
• CDW0.CID is set to a free command identifier;
• The NSID field is cleared to 0h; Submission Queues are not specific to a namespace;
• MPTR is cleared to 0h; metadata is not used for this command;
• PRP1 is set to the physical address of the PRP List. The PRP List is shown in Figure 757 for a
PRP List with three entries;
• PRP2 is cleared to 0h; PRP Entry 2 is not used for this command;
• CDW10.QSIZE is set to the size of queue to create. In this case, the value is set to 191, indicating
a queue size of 192 entries. The queue size shall not exceed the maximum queue entries
supported, indicated in the CAP.MQES field;
• CDW10.QID is set to the Submission Queue identifier;
• CDW11.CQID is set to the I/O Completion Queue identifier where command completions are
posted;
• CDW11.QPRIO is set to 10b, indicating a Medium priority queue; and
• CDW11.PC is cleared to ‘0’ indicating that the data buffer indicated by PRP1 is not physically
contiguously.
Host software then completes a transport specific action in order to submit the command for processing.
Host software shall maintain the PRP List unmodified in host memory until the Submission Queue is
deleted.
