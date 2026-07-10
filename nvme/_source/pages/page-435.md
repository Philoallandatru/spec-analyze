---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 435
headings: []
---

# Source page 435

NVM Express® Base Specification, Revision 2.1

413
Figure 460: Tracked Memory Change Data Structure
Bytes Description
01
Attributes (ATTRB): This field identifies attributes associated with the tracked memory
changes.
Bits Description
7:2 Reserved
1
Suspended (SUSP): If this bit is set to ‘1’, then the controller was suspended during
the processing of the command. If this bit is cleared to ‘0’, then the controller was
not suspended during the processing of the command.
0
More To Report (MTR):  If this bit is set to ‘1’, then the controller was not able to
report all of the tracked memory changes available to be reported in this returned
data structure. The host should issue a subsequent Track Receive command to
obtain any tracked memory changes not reported.
If this bit is cleared to ‘0’, the controller reported all of the tracked memory changes
available to be reported, if any, at the time the Track Receive command was
processed.
If the NTMCD field is cleared to 0h, then this bit shall be cleared to ‘0’.

03:02
Controller Identifier (CNTLID) : This field indicates the identifier for the controller whose
memory modifications are being tracked and reported in the Tracked Memory Changed
Descriptor list. (i.e., the same controller identifier as specified by the host in the Management
Operation Specific field (refer to Figure 459)).
07:04
Number of Tracked Memory Changed Descriptors (NTMCD): This field indicates the
number of Tracked Memory Changed Descriptors in this data structure. A value of 0h
indicates that at the time of processing the Track Receive command, there was no tracked
memory changes to report.
09:08
Reported Memory Range Granularity (RPMPG): This field specifies the granularity unit size
of the Length field in each Tracked Memory Changed Descriptor containing in this data
structure. The value specified is a power of two (2^n) times 4 KiB. For example, a value of 8h
indicates that the granularity of tracking is (2^8) * 4 KiB which is equal to 1 MiB.
15:10 Reserved
Tracked Memory Changed Descriptor List
31:16 Tracked Memory Changed Descriptor 1: This field contains the first Tracked Memory
Change Descriptor as defined in Figure 463, if any.
47:32 Tracked Memory Changed Descriptor 2: This field contains the second Tracked Memory
Changed Descriptor as defined in Figure 463, if any.
…
(NTMCD*16)+15:
(NTMCD-1)*16+16
Tracked Memory Changed Descriptor NTMCD: This field contains the last Tracked
Memory Changed Descriptor as defined in Figure 463, if any.
…
Figure 463 defines a Tracked Memory Changed Descriptor.
Figure 461: Tracked Memory Changed Descriptor
Bytes Description
07:00
Start Address (SADDR): This field indicates the 64 -bit starting address of a memory range in which one
or more bytes of data have been written. The address in this field shall be aligned to the granularity specified
by the RPMPG field. For example, if the value in the RPMPG fie ld is 8h, then the value in this field is
required to be aligned to 1 MiB (i.e., bits 19:0 are cleared to 0h).
11:08
Length (LEN): This field specifies the length of this memory range in units of the tracking granularity as
indicated in the RPMPG field. For example, if the value in the RPMPG field is 8h and the value of this field
is 4h then the length of this length of this memory range is 1 MiB * 4h which is 4 MiB.
15:12 Reserved
