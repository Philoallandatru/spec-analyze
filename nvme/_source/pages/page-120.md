---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 120
headings: ["3.3.3.4 Queue Coordination", "3.4 Command Processing", "3.4.1 Command Ordering Requirements", "3.4.2 Fused Operations"]
---

# Source page 120

NVM Express® Base Specification, Revision 2.1

98
urgent priority class arbitration mechanism is not supported, then the priority setting is not used and is
ignored by the controller.
 Queue Coordination
There is one Admin Queue pair associated with multiple I/O queue pairs. The Admin Submission Queue
and Completion Queue are used to carry out functions that impact the entire controller. An I/O Submission
Queue and Completion Queue may be used to carry out I/O (read/write) operations and may be distributed
across CPU cores and threads.
An Admin command may impact one or more I/O queue pairs. The host should ensure that Admin actions
are coordinated with threads that are responsible for the I/O queue pairs to avoid unnecessary error
conditions. The details of this coordination are outside the scope of this specification.
3.4 Command Processing
This section describes the command issue and completion mechanism. It also describes how commands
are built by host software and command completion processing.
Commands shall only be submitted by the host when the controller is ready as indicated in the Controller
Status property (CSTS.RDY) and after appropriate I/O Submission Queue(s) and I/O Completion Queue(s)
have been created.
 Command Ordering Requirements
Commands which are not part of a fused operation (refer to section 3.4.2) and which comply with atomic
operations requirements (refer to section 3.4.3), are processed as independent entit ies without reference
to other commands submitted to the same I/O Submission Queue or to commands submitted to other I/O
Submission Queues. For example , the controller is not responsible for checking the LBA of an NVM
Command Set Read command or Write command to ensure any type of ordering between commands. If a
Read command is submitted for LBA x and there is a Write command also submitted for LBA x, there is no
guarantee of the order of completion for those commands (the Read command may finish first or the Write
command may finish first). If there are ordering requirements between these commands, host software or
the associated application is required to enforce that ordering above the level of the controller.
 Fused Operations
Fused operations enable a more complex command by “fusing” together two simpler commands. This
feature is optional; support for this feature is indicated in  the FUSES field in the Identify Controller data
structure in Figure 312.
Whether a command is part of a fused operation is specified by the Fused Operation (FUSE) field of
Command Dword 0 shown in Figure 91. The FUSE field also specifies whether the command is the first
command in the fused operation or the second command in the fused operation. If the FUSE field is set to
a non-zero value and the controller does not support the requested fused operation, th en the controller
should abort the command with a status code of Invalid Field in Command.
In a fused operation, the requirements are:
• The commands shall be executed in sequence as an atomic unit. The controller shall behave as if
no other operations have been executed between these two commands;
• The operation ends at the point an error is encountered in either command. If the first command in
the sequence failed, then the second command in the sequence shall be aborted. If the second
command in the sequence failed, then the completion status of the first command is sequence
specific and is defined within the Fused Operation section of the applicable I/O Command Set
specification;
• The commands shall be inserted next to each other in the same Submission Queue. If the controller
processes a command violating this condition (e.g., a command with the FUSE field cleared to 00b
(i.e., Normal operation) is inserted immediately after a comm and specifying the FUSE field set to
01b (i.e., Fused operation, first command), or is inserted immediately before a command specifying
