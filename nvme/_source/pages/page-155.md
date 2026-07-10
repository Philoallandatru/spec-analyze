---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 155
headings: ["4 Data Structures", "4.1 Submission Queue Entry", "4.1.1 Admin Command and I/O Command Common SQE"]
---

# Source page 155

NVM Express® Base Specification, Revision 2.1

133
4 Data Structures
This section describes data structures used by the NVM Express Interface.
4.1 Submission Queue Entry
 Admin Command and I/O Command Common SQE
Each Common Command Format command is 64 bytes in size.
Command Dword 0, Namespace Identifier, Metadata Pointer, PRP Entry 1, PRP Entry 2, SGL Entry 1, and
Metadata SGL Segment Pointer have common definitions for all Admin commands and I/O commands for
all I/O Command Sets. Metadata Pointer, PRP Entry 1, PRP En try 2, and Metadata SGL Segment Pointer
are not used by all commands. Command Dword 0 is defined in Figure 91.
Figure 91: Command Dword 0
Bits Description
31:16
Command Identifier (CID): This field specifies a unique identifier for the command when combined with the
Submission Queue identifier.
The value of FFFFh should not be used as the Error Information log page (refer to section 5.1.12.1.2) uses
this value to indicate an error is not associated with a particular command.
15:14
PRP or SGL for Data Transfer (PSDT):  This field specifies whether PRPs or SGLs are used for any data
transfer associated with the command. PRPs shall be used for all Admin commands for NVMe over PCIe
implementations. SGLs shall be used for all Admin and I/O commands for NVMe over Fabrics implementations
(i.e., this field set to 01b). An NVMe Transport may support only specific values (refer to the applicable NVMe
Transport binding specification for details).
Value Definition
00b PRPS Used: PRPs are used for this transfer.
01b
SGLs Used MPTR Address: SGLs are used for this transfer. If used, Metadata Pointer
(MPTR) contains an address of a single contiguous physical buffer.
Refer to the Metadata Buffer Alignment (MBA) bit of the SGLS field in the Identify
Controller data structure (refer to Figure 312) for alignment requirements.
10b
SGLs Used MPTR SGL Segment:  SGLs are used for this transfer. If used, Metadata
Pointer (MPTR) contains an address of an SGL segment containing exactly one SGL
Descriptor that is qword aligned.
11b Reserved
If there is metadata that is not interleaved with the user data, as specified in the Format NVM command, then
the Metadata Pointer (MPTR) field is used to point to the metadata. The definition of the Metadata Pointer field
is dependent on the setting in this field. Refer to Figure 92.
13:10 Reserved
09:08
Fused Operation (FUSE): In a fused operation, a complex command is created by “fusing” together two
simpler commands. Refer to section 3.4.2. This field specifies whether this command is part of a fused
operation and if so, which command it is in the sequence.
Value Definition
00b Normal operation
01b First command of Fused operation
10b Second command of Fused operation
11b Reserved
