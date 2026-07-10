---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 427
headings: ["5.1.25.2 Memory-Based Transport Feature Specific Information (PCIe)", "5.1.25.2.1 Number of Queues (Feature Identifier 07h)"]
---

# Source page 427

NVM Express® Base Specification, Revision 2.1

405
Figure 442: Boot Partition Write Protection Config - Command Dword 11
Bits Description
02:00
Boot Partition 0 Write Protection State  (BP0WPS): This field specifies the write protection state of
Boot Partition 0.
The default value of this field is Write Locked.
Value Definition
000b1 Change in state not requested
001b Write Unlocked
010b Write Locked
011b Write Locked Until Power Cycle
100b2 Write Protection controlled by RPMB
101b to 111b Reserved

Notes:
1. A value of 000b is not returned by a controller via a Get Features command. This value is only used in the Set
Features command.
2. A value of 100b is reserved for the Set Features command for both the Boot Partition 0 Write Protection State
field and the Boot Partition 1 Write Protection State field. The Boot Partition write protection states transition to
this state when RPMB Boot Partition Write Protection is enabled (refer to section 8.1.3.3.2 and section 8.1.21).
If a Set Features command is submitted that attempts to change the Boot Partition write protection state of
a Boot Partition that is in the Write Locked Until Power Cycle state, then the controller shall abort the
command with a status code of Feature Not Changeable.
If a Set Features command is submitted that attempts to change either Boot Partition write protection state
from a value of 100b (i.e., Boot Partition write protection is controlled by RPMB), then the controller shall
abort the command with a status code of Feature Not Changeable.
If a Set Features command attempts to change the Boot Partition write protection state of a Boot Partition
shared across multiple controllers to the Write Locked Until Power cycle state in a multi -domain NVM
subsystem (i.e., the MDS bit is set to ‘1’ in the CTRATT field of the Identify Controller data structure (refer
to Figure 312), then the controller shall abort the command with a status code of Feature Not Changeable.
 Memory-Based Transport Feature Specific Information (PCIe)
5.1.25.2.1 Number of Queues (Feature Identifier 07h)
This Feature indicates the number of queues that the host requests for the controller processing the
command. This Feature shall only be issued during initialization prior to creation of any I/O Submission
and/or Completion Queues. If a Set Features comman d is issued for this Feature after creation of any I/O
Submission and/or I/O Completion Queues, then the Set Features command shall abort with status code
of Command Sequence Error. The controller shall not change the value allocated between resets. For a
Set Features command, the attributes are specified in Command Dword 11 (refer to Figure 443). For a Get
Features command, Dword 11 is ignored.
If a Set Features or Get Features command is submitted for this Feature, the attributes specified in Figure
444 are returned in Dword 0 of the completion queue entry for that command.
Figure 443: Number of Queues – Command Dword 11
Bits Description
31:16
Number of I/O Completion Queues Requested (NCQR): Indicates the number of I/O Completion
Queues requested by software. This number does not include the Admin Completion Queue. A minimum
of one queue shall be requested, reflecting that the minimum support is for one I/O Completion Queue.
This is a 0’s based value. The maximum value that may be specified is 65,534 (i.e., 65,535 I /O
Completion Queues). If the value specified is 65,535, the controller should abort the command with a
status code of Invalid Field in Command.
