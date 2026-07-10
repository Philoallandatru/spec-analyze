---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 180
headings: []
---

# Source page 180

NVM Express® Base Specification, Revision 2.1

158
of the logical blocks accessed on the controller is 13  KiB. There are three SGL segments describing the
locations in memory where the logical block data is transferred.
The three SGL segments contain a total of three Data Block descriptors with lengths of 3  KiB, 4 KiB, and
4 KiB respectively. Segment 1 of the Destination SGL contains a Bit Bucket descriptor with a length of 2 KiB
that specifies to not transfer (i.e., igno re) 2  KiB of logical block data from the NVM. Segment 1 of the
destination SGL also contains a Last Segment descriptor specifying that the segment pointed to by the
descriptor is the last SGL segment.
Figure 123: SGL Read Example
3KiB
 4KiB
 2KiB
 4KiB
Bit Bucket data
not transfered
LBA x+26NVMe Logical Blocks
Data Block A
 Data Block C
 Data Block B
Host DRAM
SGL Data Block descriptor
specifies to transfer 3KiB
Segment descriptor
points to the next memory
location in the SGL
Bit Bucket descriptor specifies
to not transfer the next 2KiB of
logical block data
Data Block descriptor
specifies to transfer 4KiB
Last Segment descriptor
points to the last memory
location in the SGL
Data Block descriptor
specifies to transfer 4KiB
Address = A
0
 Length = 3KiB
Address = Segment 1
2
 Length = 48
Address = B
0
 Length = 4KiB
1
 Length = 2KiB
Address = Segment 2
3
 Length = 16
Address = C
0
 Length = 4KiB
Destination SGL Segment 2
Destination SGL Segment 1
Destination SGL Segment 0
LBA x
