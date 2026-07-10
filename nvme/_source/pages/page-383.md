---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 383
headings: ["5.1.17.2 Command Completion", "5.1.18 NVMe-MI Receive command", "5.1.19 NVMe-MI Send command", "5.1.20 Namespace Attachment command"]
---

# Source page 383

NVM Express® Base Specification, Revision 2.1

361
If the CSVI field(refer to Figure 354) is cleared to 0h and the CSUUIDI field (refer to Figure 354) is cleared
to 0h, then the controller shall abort the command with a status code of Invalid Field in Command.
If the Migration Send command for this management operation specifies the SEQIND field set to 10b and
that command completes successfully, then the controller state shall be verified and committed to the
specified controller which includes:
• Creating any I/O Submission Queues and I/O Completion Queues specified by the NVMe
Controller State field; and
• setting the queue state for each I/O Submission Queues and I/O Completion Queues specified by
the NVMe Controller State field.
If any Migration Send command for this management operation is not successful, then the host should
attempt to re-transfer the entire controller state data so that any existing controller state is overwritten with
the desired controller state.
 Command Completion
Upon completion of the Migration Send command, the controller posts a completion queue entry to the
Admin Completion Queue indicating the status for the command. Section  5.1.17.1 describes completion
details for each management operation.
Migration Send command specific status values (i.e., SCT field set to 1h) are shown in  Figure 362.
Figure 362: Migration Send – Command Specific Status Values
Value Definition
1Fh Invalid Controller Identifier: The specified controller is not in a condition to set the controller state.
37h Invalid Controller Data Queue:  This error indicates that the specified Controller Data Queue
Identifier is invalid for the controller processing the command.
38h Not Enough Resources: This error indicates that there is not enough resources in the controller to
process the command.
3Ah Controller Not Suspended: The operation requested is not allowed if the specified controller is not
suspended (refer to section 5.1.17.1.1).
 NVMe-MI Receive command
Refer to the NVM Express Management Interface Specification for details on the NVMe -MI Receive
command.
 NVMe-MI Send command
Refer to the NVM Express Management Interface Specification for details on the NVMe-MI Send command.
 Namespace Attachment command
The Namespace Attachment command is used to attach and detach controllers from a namespace.  The
attach and detach operations (refer to section 8.1.15) are persistent across all reset events.  Namespace
attach and detach operations are persistent across Virtualization Management commands that set a
secondary controller offline.
The Namespace Attachment command uses  the Data Pointer and Command Dword 10 field s. All other
command specific fields are reserved.
The Select field determines the data structure used as part of the command. The data structure is 4 ,096
bytes in size. The data structure used for Controller Attach and Controller Detach is a Controller List (refer
to section 4.6.1). The controllers that are to be attached or detached, respectively, are described in the data
structure.
If the SEL field specifies the Controller Attach value, then
• If the Maximum Domain Namespace Attachments (MAXDNA) field in the Identify Controller data
structure (refer to Figure 312) is non-zero, then:
