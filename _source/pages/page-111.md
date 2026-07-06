---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 111
headings: ["3.3.2.1.1 Command Capsules", "3.3.2.1.2 Response Capsules", "3.3.2.1.3 Data Transfers"]
---

# Source page 111

NVM Express® Base Specification, Revision 2.1

89
3.3.2.1.1 Command Capsules
A command capsule is sent from a host to a controller. It contains a submission queue entry (SQE) and
may optionally contain data or SGLs. The SQE is 64 bytes in size and contains the Admin command, I/O
command, or Fabrics command to be executed.
Figure 75: Command Capsule

Submission Queue Entry Data or SGLs (if present)
Byte 0 63 64 (N-1)
Command Capsule of Size N Bytes
The Command Identifier field in the SQE shall be unique among all outstanding commands associated with
that queue. If there is data or additional SGLs to be transferred within the capsule, then the SGL descriptor
in the SQE contains a Data Block, Segment Descriptor, or Last Segment Descriptor specifying an
appropriate Offset address. The definition for the submission queue entry when the command is a Fabrics
command is shown in section 4.1.2. The definition for the submission queue entry when the command is
an Admin command or I/O command is shown in section 4.1.1. Bytes 03:00 share a common format across
commands.
3.3.2.1.2 Response Capsules
A response capsule is sent from the NVM subsystem to the host. It contains a completion queue entry
(CQE) and may optionally contain data. The CQE is the completion queue entry associated with a
previously issued command capsule.
If a command requests data and the SGL in the associated command capsule specifies a Data Block
descriptor with an Offset, the data is included in the response capsule. If the SGL(s) in the command
capsule specify a region in host memory, then data is transferred via memory transactions.
Figure 76: Response Capsule

Completion Queue Entry Data (if present)
Byte 0 15 16 (N-1)
Response Capsule of Size N Bytes
The completion queue entry is 16 bytes in size and contains a two byte status field.
The definition for the completion queue entry for a Fabrics command is shown in section 4.2.2. The
definition for the completion queue entry when the command is an Admin command or I/O command is
defined in section 4.2.1, where the SQ Identifier and Phase Tag fields are reserved because they are not
used in NVMe over Fabrics. Use of the SQHD field depends on whether SQ flow control is disabled for the
queue pair, refer to section 6.3.
3.3.2.1.3 Data Transfers
Data may be transferred within capsules or by memory transfers. SGLs are used to specify the location of
data. Metadata, if transferred, is a contiguous part of the user data with which that metadata is associated.
The SGL descriptor(s) (refer to section 4.3.2) specify whether the command’s data is transferred through
memory or within the capsule. The capsule may contain either SGLs or data (not a mixture of both) following
