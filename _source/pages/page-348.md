---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 348
headings: []
---

# Source page 348

NVM Express® Base Specification, Revision 2.1

326
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
575 O O P
Maximum Memory Range Tracking Granularity (MAXMRTG): This field indicates the
maximum value supported in the Requested Memory Range Tracking Granularity
(RMRTG) field (refer to Figure 469) of the Track Memory Ranges data structure.
If the THMCS bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should
ignore this field.
576 O O P
Tracking Attributes (TRATTR): This field indicates supported attributes for the Track
Send command and Track Receive command.

Bits Description
7:3 Reserved
2
Memory Range Tracking Length Limit (MRTLL): If this bit is set to ‘1’, then
the controller only supports the length as specified by the Requested Memory
Range Tracking Granularity field and the Length field in a Track Memory
Changes data structure (refer to Figure 469) indicating a value that is a power
of 2. If this bit is cleared to ‘0’, then the length indicated by the Requested
Memory Range Tracking Granularity field and the Length field in a Track
Memory Changes data structure is not required to be a value that is a power
of 2.
1
Track User Data Changes Support (TUDCS): If this bit is set to ‘1’, then the
controller supports tracking user data changes as defined by section
5.1.26.1.1. If this bit is cleared to ‘0’, then the controller does not support
tracking user data changes as defined by section 5.1.26.1.1.
0
Track Host Memory Changes Support (THMCS): If this bit is set to ‘1’, then
the controller supports tracked memory changes as defined by section
5.1.26.1.1 and section 5.1.27.1.2. If this bit is cleared to ‘0’, then the controller
does not support the tracked memory changes as defined by section
5.1.26.1.1 and section 5.1.27.1.2.
577    Reserved
579:578 O O P
Maximum Controller User Data Migration Queues (MCUDMQ): This field indicates the
maximum number of User Data Migration Queues supported by the controller (i.e.,
allowed to be created in this controller).
If the TUDCS bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should
ignore this field.
581:580 O O P
Maximum NVM Subsystem User Data Migration Queues (MNSUDMQ) : This field
indicates the maximum number of User Data Migration Queues supported by the NVM
subsystem (i.e., allowed to be created in this NVM subsystem).
If the TUDCS bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should
ignore this field.
583:582 O O P
Maximum CDQ Memory Ranges (MCMR): This field indicates the maximum number of
memory ranges allowed to be specified by the PRP1 field of a Controller Data Queue
command. A value of 0h indicates that a maximum is not reported.
585:584 O O P
NVM Subsystem Maximum CDQ Memory Ranges ( NMCMR): This field indicates the
maximum number of memory ranges for all Controller Data Queues in the NVM
subsystem. A value of 0h indicates that a maximum is not reported.
The value of this field shall be greater than or equal to the value of the MCMR field.
587:586 O O P
Maximum Controller Data Queue PRP Count (MCDQPC): This field indicates the
maximum number of PRPs allowed to be specified in the PRP list in the Controller Data
Queue command. A value of 0h indicates that a maximum is not reported.
767:588    Reserved
