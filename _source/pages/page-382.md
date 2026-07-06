---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 382
headings: []
---

# Source page 382

NVM Express® Base Specification, Revision 2.1

360
Figure 360: I/O Submission Queue State Data Structure
Bytes Description
15:14
I/O Submission Queue Attributes (IOSQA): This field contains various attributes associated with the I/O
Submission Queue.

Bits Description
15:03 Reserved
02:01
I/O Submission Queue Priority (IOSQQPRIO): This field contains the contents of the QPRIO
field (refer to Figure 479) from the Create I/O Submission Queue command that created this
I/O Submission Queue.
00
I/O Submission Queue Physically Contiguous (IOSQPC): This bit contains the contents of
the PC bit (refer to Figure 479) from the Create I/O Submission Queue command that created
this I/O Submission Queue.
17:16 I/O Submission Queue Head Pointer (IOSQHP): This field indicates the value of the I/O Submission
Queue head pointer.
19:18 I/O Submission Queue Tail Pointer (IOSQTP): This field indicates the value of the I/O Submission
Queue tail pointer.
23:20 Reserved

Figure 361: I/O Completion Queue State Data Structure
Bytes Description
07:00
I/O Completion Queue Size  PRP Entry 1 ( IOCQPRP1): This field contains the contents of the PRP1
field (refer to  Figure 473) from the Create I/O Completion Queue command that created this I/O
Completion Queue.
09:08 I/O Completion Queue Size (IOCQQSIZE): This field contains the contents of the QSIZE field (refer to
Figure 474) from the Create I/O Completion Queue command that created this I/O Completion Queue.
11:10 I/O Completion Queue Identifier (IOCQQID): This field contains the contents of the PRP1 field (refer to
Figure 474) from the Create I/O Completion Queue command that created this I/O Completion Queue.
13:12 I/O Completion Queue Head Pointer (IOCQHP): This field indicates the value of the I/O Completion
Queue head pointer.
15:14 I/O Completion Queue Tail Pointer (IOCQTP): This field indicates the value of the I/O Completion
Queue tail pointer.
19:16
I/O Completion Queue Attributes (IOCQA): This field contains various attributes associated with the
I/O Completion Queue.

Bits Description
31:16
I/O Completion Queue Interrupt Vector (IOCQIV): This field contains the contents of the
IV field (refer to Figure 475) from the Create I/O Completion Queue command that created
this I/O Completion Queue.
15:03 Reserved
02 Slot 0 Phase Tag (S0PT): This bit states the value of the Phase Tag bit for slot 0 of this I/O
Completion Queue.
01
I/O Completion Queue Interrupts Enabled (IOCQIEN): This bit contains the contents of
the IEN bit (refer to Figure 475) from the Create I/O Completion Queue command that
created this I/O Completion Queue.
00
I/O Completion Queue Physically Contiguous (IOCQPC): This bit contains the contents
of the PC bit (refer to Figure 475) from the Create I/O Completion Queue command that
created this I/O Completion Queue.
23:20 Reserved
If the CSVI field (refer to Figure 354) is cleared to 0h and the NVMe Controller State Size field (refer to
Figure 358) is not cleared to 0h, then the controller shall abort the command with a status code of Invalid
Field in Command.
If the CSUUIDI field (refer to Figure 354) is cleared to 0h and the Vendor Specific Size field (refer to Figure
358) is not cleared to 0h, then the controller shall abort the command with a status code of Invalid Field in
Command.
