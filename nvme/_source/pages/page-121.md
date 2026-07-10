---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 121
headings: ["3.4.3 Atomic Operations", "3.4.4 Command Arbitration"]
---

# Source page 121

NVM Express® Base Specification, Revision 2.1

99
the FUSE field set to 10b (i.e., Fused operation, second command)), then the controller shall abort
the command specifying non-zero values of the FUSE field with a status code of Command Aborted
due to Missing Fused Command. If the first command is in the last slot in the Submission Queue,
then the second command shall be in the first slot in the Submission Queue as part of wrapping
around. In the memory-based transport queue model, the Submission Queue Tail doorbell pointer
update shall indicate both comma nds as part of one doorbell update. In the message -based
transport queue model, the command capsules shall be submitted in-order;
• To abort the fused operation, the host submit s an Abort command separately for each of the
commands; and
• A completion queue entry is posted by the controller for each of the commands.
Refer to each I/O Command Set specification for applicability and additional details, if any.
 Atomic Operations
The definition for atomic operations is command set specific. Refer to each I/O Command Set specification
for applicability and additional details, if any.
 Command Arbitration
After a command has been  submitted to the controller (refer to section 1.5.19), the controller transfers
submitted commands into the controller for subsequent processing using a vendor specific algorithm.
A command is being processed when the controller and/or namespace state is being accessed or modified
by the command such as:
• a Feature setting is being accessed;
• a Feature setting is being modified;
• user data (e.g., a logical block as defined by the NVM Command Set Specification) is being
accessed; or
• user data is being modified.
A command is completed when a completion queue entry for the command has been posted to the
corresponding Completion Queue. Upon completion, all controller state and/or namespace state
modifications made by that command are globally visible to all subsequently submitted commands.
A candidate command is a submitted command which has been transferred into the controller that the
controller deems ready for processing. The controller selects command(s) for processing from the pool of
submitted commands for each Submission Queue. The co mmands that comprise a fused operation shall
be processed together and in order by the controller. The controller may select candidate commands for
processing in any order. The order in which commands are selected for processing does not imply the order
in which commands are completed.
Arbitration is the method used to determine the Submission Queue from which the controller starts
processing the next candidate command(s). Once a Submission Queue is selected using arbitration, the
Arbitration Burst setting determines the maximum number o f commands that the controller may start
processing from that Submission Queue before arbitration shall again take place. A fused operation may
be considered as one or two commands by the controller.
All controllers shall support the round robin command arbitration mechanism. A controller may optionally
implement weighted round robin with urgent priority class and/or a vendor specific arbitration mechanism.
The Arbitration Mechanism Supported field in the Controller Capabilities property (CC.AMS) indicates
optional arbitration mechanisms supported by the controller.
In order to make efficient use of the non -volatile memory, it is often advantageous to execute multiple
commands from a Submission Queue in parallel. For Submission Queues that are using weighted round
robin with urgent priority class or round robin arbitr ation, host software may configure an Arbitration Burst
setting. The Arbitration Burst setting indicates the maximum number of commands that the controller may
launch at one time from a particular Submission Queue. It is recommended that host software conf igure
the Arbitration Burst setting as cl ose to the recommended value by the controller as possible (specified in
