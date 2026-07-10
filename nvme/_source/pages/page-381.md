---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 381
headings: []
---

# Source page 381

NVM Express® Base Specification, Revision 2.1

359
Figure 359: NVMe Controller State Data Structure
Bytes Description
Header
01:00 Version (VER): This field indicates the version of this data structure. This field shall be
cleared to 0h.
03:02 Number of I/O Submission Queues (NIOSQ):  This field indicates the number I/O
Submission Queues contained in this data structure.
05:04 Number of I/O Completion Queues (NIOCQ):  This field indicates the number I/O
Completion Queues contained in this data structure.
07:06 Reserved
I/O Submission Queue List
31:08
I/O Submission Queue State 0: This field contains the state of the first I/O Submission
Queue reported, if any. The contents of this field are defined in the I/O Submission Queue
State data structure (refer to Figure 360).
55:32
I/O Submission Queue State 1: This field contains the state of the second I/O
Submission Queue reported, if any. The contents of this field are defined in the I/O
Submission Queue State data structure (refer to Figure 360).
…
((NIOSQ) * 24) + 7:
((NIOSQ-1) * 24) + 8
I/O Submission Queue State NIOSQ -1: This field contains the state of the last I/O
Submission Queue reported, if any. The contents of this field are defined in the I/O
Submission Queue State data structure (refer to Figure 360).
I/O Completion Queue List
 ((NIOSQ+1) * 24) + 7:
((NIOSQ) * 24) + 8
I/O Completion Queue State 0: This field contains the state of the first I/O Completion
Queue reported, if any. The contents of this field are defined in the I/O Completion Queue
State data structure (refer to Figure 361).
((NIOSQ+2) * 24) + 7:
((NIOSQ+1) * 24) + 8
I/O Completion Queue State 1: This field contains the state of the second I/O
Completion Queue reported, if any. The contents of this field are defined in the I/O
Completion Queue State data structure (refer to Figure 361).
…
((NIOCQ) * 24) +
((NIOSQ) * 24) + 15:
((NIOCQ-1) * 24) +
((NIOSQ) * 24) + 8
I/O Completion Queue State NIOSQ -1: This field contains the state of the last I/O
Completion Queue reported, if any. The contents of this field are defined in the I/O
Completion Queue State data structure (refer to Figure 361).
The I/O Submission Queue list shall be listed in ascending order by I/O Submission Queue Identifier.
The I/O Completion Queue list shall be listed in ascending order by I/O Completion Queue Identifier.
Figure 360: I/O Submission Queue State Data Structure
Bytes Description
07:00 I/O Submission PRP Entry 1 (IOSQPRP1): This field contains the contents of the PRP1 field (refer to
Figure 477) from the Create I/O Submission Queue command that created this I/O Submission Queue.
09:08 I/O Submission Queue Size ( IOSQQSIZE): This field contains the contents of the QSIZE field (refer to
Figure 478) from the Create I/O Submission Queue command that created this I/O Submission Queue.
11:10 I/O Submission Queue Identifier ( IOSQQID): This field contains the contents of the QID field (refer to
Figure 478) from the Create I/O Submission Queue command that created this I/O Submission Queue.
13:12 I/O Completion Queue Identifier ( IOSQCQID): This field contains the contents of the CQID field (refer
to Figure 479) from the Create I/O Submission Queue command that created this I/O Submission Queue.
