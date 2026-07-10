---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 113
headings: ["3.3.2.1.3.2 Data Transfer Examples"]
---

# Source page 113

NVM Express® Base Specification, Revision 2.1

91
Figure 77: Data and SGL Locations within a Command Capsule

Submission Queue Entry Additional SGLs
(if present)
Byte 0 63 64
Command Capsule of Size N Bytes
(ICDOFF * 16) + 64
Submission Queue Entry Data (if present)Undefined
(if ICDOFF > 0)
Byte 0 63 64 (N-1)
Command Capsule of Size N Bytes
Undefined
Undefined
(M-1)
(N-1)(M-1)

 Data Transfer Examples
The data transfer examples in Figure 78 and Figure 79 show SGL examples for a Write command where
data is transferred via a memory transaction or within the capsule. The SGL may use a key as part of the
data transfer depending on the requirements of the NVMe Transport used.
The first example shows an 8KiB write where all of the data is transferred via memory transactions. In this
case, there is one SGL descriptor that is contained within the submission queue entry at CMD.SGL1. The
SGL descriptor is a Keyed SGL Data Block descriptor. If more SGLs are required to complete the command,
the additional SGLs are contained in the command capsule.
Figure 78: SGL Example Using Memory Transactions

Host DRAM
Data Block A
SGL Descriptor
Address = Data Block A
Keyed Data Block descriptor
specifies to transfer 8KiB
through memory
Length = 8KiB
SGL Identifier = 40h
Key = Tag A
