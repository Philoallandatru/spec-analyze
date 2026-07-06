---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 107
headings: ["3.3 NVM Queue Models", "3.3.1 Memory-based Transport Queue Model (PCIe)", "3.3.1.1 Queue Setup and Initialization", "3.3.1.2 Queue Usage"]
---

# Source page 107

NVM Express® Base Specification, Revision 2.1

85
• host data redundancy software (e.g., RAID) that may use the Endurance Group’s Domain Identifier
to determine which Endurance Groups may fail together (e.g., Endurance Groups in the same
domain) and which Endurance Groups may fail independently (e.g., Endurance Groups in different
domains); and
• host application software may use the controller’s Domain Identifier to determine which controllers
share domains (e.g., controllers that may fail together) and which controllers are a part of different
domains (e.g., controllers that may fail independently).
3.3 NVM Queue Models
The NVM Express interface is based on a paired Submission and Completion Queue mechanism.
Commands are placed by host software into a Submission Queue. Completions are placed into the
associated Completion Queue by the controller. When using a memory-based transport queue model (refer
to section 3.3.1), m ultiple Submission Queues may utilize the same Completion Queue.  When using a
message-based transport queue model (refer to section 3.3.2) each Submission Queue maps to a single
Completion Queue.
 Memory-based Transport Queue Model (PCIe)
 Queue Setup and Initialization
To setup and initialize I/O Submission Queues and I/O Completion Queues for use, host software follows
these steps:
1. Configures the Admin Submission Queue and the Admin Completion Queue s by initializing the
Admin Queue Attributes (AQA), Admin Submission Queue Base Address (ASQ), and Admin
Completion Queue Base Address (ACQ) properties appropriately;
2. Configures the size of the I/O Submission Queues (CC.IOSQES) and I/O Completion Queues
(CC.IOCQES);
3. Submits a Set Features command with the Number of Queues attribute set to the requested
number of I/O Submission Queues and I/O Completion Queues. The completion queue entry for
this Set Features command indicates the number of I/O Submission Queues and I/ O Completion
Queues allocated by the controller;
4. Determines the maximum number of entries supported per queue (CAP.MQES) and whether the
queues are required to be physically contiguous (CAP.CQR);
5. Creates I/O Completion Queues within the limitations of the number allocated by the controller and
the queue attributes supported (maximum entries and physically contiguous requirements) by using
the Create I/O Completion Queue command; and
6. Creates I/O Submission Queues within the limitations of the number allocated by the controller and
the queue attributes supported (maximum entries and physically contiguous requirements) by using
the Create I/O Submission Queue command.
At the end of this process, I/O Submission Queues and I/O Completion Queues have been setup and
initialized and may be used to complete I/O commands.
 Queue Usage
The submitter of entries to a memory-based transport queue uses the current Tail entry pointer to identify
the next open queue slot. The submitter increments the Tail entry pointer after placing the new entry to the
open queue slot. If the Tail entry pointer increment exceeds the queue size, the Tail entry shall roll to zero.
The submitter may continue to place entries in free queue slots as long as the Full queue condition is not
met (refer to section 3.3.1.5).
Note: The submitter shall take queue wrap conditions into account.
The consumer of entries on a memory-based transport queue uses the current Head entry pointer to identify
the slot containing the next entry to be consumed. The consumer increments the Head entry pointer after
consuming the next entry from the queue. If the Head entry pointer increment exceeds the queue size, the
Head entry pointer shall roll to zero. The consumer may continue to consume entries from the queue as
long as the Empty queue condition is not met (refer to section 3.3.1.4).
