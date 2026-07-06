---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 340
headings: []
---

# Source page 340

NVM Express® Base Specification, Revision 2.1

318
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
NVM Command Set Attributes
512 M M R
Submission Queue Entry Size (SQES): This field defines the required and maximum
I/O Submission Queue entry size.
Bits Description
7:4
Maximum I/O Submission Queue Entry Size  (MAXSQES): T his field
identifies the maximum I/O Submission Queue entry size when using the NVM
Command Set. This value is greater than or equal to the required SQ entry size
(i.e., the MIOSQES field). The value is in bytes and is reported as a power of
two (2^n). The recommended value is 6, corresponding to a standard SQ entry
size of 64 bytes. Controllers that implement proprietary extensions may support
a larger value.
3:0
Minimum I/O Submission Queue Entry Size (MINSQES): This field identifies
the required (i.e., minimum) I/O Submission Queue entry size. This is the
minimum entry size that may be used. The value is in bytes and is reported as
a power of two (2^n). The required value shall be 6, corresponding to 64.

513 M M R
Completion Queue Entry Size (CQES): This field defines the required and maximum
I/O Completion Queue entry size.
Bits Description
7:4
Maximum I/O Completion Queue Entry Size (MAXCQES): T his field
identifies the maximum I/O Completion Queue entry size. This value is greater
than or equal to the required CQ entry size (i.e., the MIOCQES field). The
value is in bytes and is reported as a power of two (2^n). The recommended
value is 4, corresponding to a standard CQ entry size of 16 bytes. Controllers
that implement proprietary extensions may support a larger value.
3:0
Minimum I/O Completion Queue Entry Size (MINCQES): This field
identifies the required (i.e., minimum) I/O Completion Queue entry size. This
is the minimum entry size that may be used. The value is in bytes and is
reported as a power of two (2^n). The required value shall be 4, corresponding
to 16.

515:514 M M M
Maximum Outstanding Commands (MAXCMD): Indicates the maximum number of
commands that the controller processes at one time for a particular queue (which may be
larger than the size of the corresponding Submission Queue). The host may use this
value to size Completion Queues and optimize the number of commands submitted at
one time to a particul ar I/O Queue. This field is mandatory for NVMe over Fabrics
implementations and optional for NVMe over PCIe implementations. If the field is not
used, it shall be cleared to 0h.
519:516 M M R
Number of Namespaces (NN): This field indicates the maximum value of a valid NSID
for the NVM subsystem.  Refer to the MNAN field for the number of supported
namespaces in the NVM subsystem.
