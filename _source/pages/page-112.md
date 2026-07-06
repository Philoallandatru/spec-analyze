---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 112
headings: ["3.3.2.1.3.1 Data and SGL Locations within a Command Capsule"]
---

# Source page 112

NVM Express® Base Specification, Revision 2.1

90
the SQE. If additional SGLs are required, then the SGLs are included in the capsule immediately after the
SQE. If an invalid offset is specified in an SGL descriptor, then a status code of SGL Offset Invalid shall be
returned.
SGLs shall be supported within a capsule. The NVMe Transport binding specification defines the SGL
Descriptor Types and Sub Types that are supported for the corresponding NVMe Transport. The NVMe
Transport binding specification also specifies if SGLs may be supported in host memory.
 Data and SGL Locations within a Command Capsule
The submission queue entry within the command capsule includes one SGL entry. If there are additional
SGL entries to be transferred in the command capsule, then those entries shall be contiguous and located
immediately after the submission queue entry.
An NVMe Transport binding specification defines the support for data as part of the command capsule. The
controller indicates the starting location of data within a command capsule via the In Capsule Data Offset
(ICDOFF) field in the Identify Controller data structure.
There are restrictions for SGLs that the host should follow:
• if the ICDOFF field is a non-zero value, then all SGL descriptors following the submission queue
entry shall not have a total size greater than (ICDOFF * 16);
• if the SGL descriptors following the submission queue entry have a total size greater than (ICDOFF
* 16), then the controller shall abort the command with a status code of Invalid Number of SGL
Descriptors;
• the host shall not place more SGL Data Block or Keyed SGL Data Block descriptors within a
capsule than the maximum indicated in the Identify Controller data structure; and
• if the host places more SGL Data Block of Keyed SGL Data Block descriptors in a capsule than the
maximum indicated in the Maximum SGL Data Block Descriptors field in the Identify Controller data
structure, then the controller shall abort the command with a status code of Invalid Number of SGL
Descriptors.
The host shall start data (if present) in command capsules at byte offset (ICDOFF * 16) from the end of the
submission queue entry.
