---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 442
headings: ["5.2.1.1 Command Completion", "5.2.2 Create I/O Submission Queue command"]
---

# Source page 442

NVM Express® Base Specification, Revision 2.1

420
 Command Completion
If the command is completed, then the controller shall post a completion queue entry to the Admin
Completion Queue indicating the status for the command.
Create I/O Completion Queue command specific status values are defined in Figure 476.
Figure 476: Create I/O Completion Queue – Command Specific Status Values
Value Description
1h
Invalid Queue Identifier: The creation of the I/O Completion Queue failed due to an invalid queue identifier
specified as part of the command. An invalid queue identifier is one that identifies the Admin Queue (i.e.,
0h), is outside the range supported by the controller, or is a Completion Queue Identifie r that is already in
use.
2h
Invalid Queue Size: The host attempted to create an I/O Completion Queue:
• with an invalid number of entries (e.g., a value of 0h or a value which exceeds the maximum
supported by the controller, specified in CAP.MQES); or
• before initializing the CC.IOCQES field.
8h Invalid Interrupt Vector: The creation of the I/O Completion Queue failed due to an invalid interrupt vector
specified as part of the command.
 Create I/O Submission Queue command
The Create I/O Submission Queue command is used to create I/O Submission Queues. The Admin
Submission Queue is created by specifying its base address in the ASQ property. If a PRP List is provided
to describe the SQ to be created, then the PRP List shall b e maintained by host software at the same
location in host physical memory and the values in the PRP List shall not be modified until the corresponding
Delete I/O Submission Queue command for that SQ is completed or the controller is reset. If the PRP List
values are modified, the behavior is undefined.
The Create I/O Submission Queue command uses the PRP Entry 1, Command Dword 10, Command
Dword 11, and Command Dword 12 fields. All other command specific fields are reserved.
Figure 477: Create I/O Submission Queue – PRP Entry 1
Bits Description
63:00
PRP Entry 1 (PRP1): If CDW11.PC is set to ‘1’, then this field specifies a 64 -bit base memory address
pointer of the Submission Queue that is physically contiguous. The address pointer is memory page
aligned (based on the value in CC.MPS) unless otherwise specified. If CDW11.PC is cleared to ‘0’, then
this field specifies a PRP List pointer that describes the list of pages that constitute the Su bmission
Queue. The list of pages is memory page aligned (based on the value in CC.MPS) unless otherwise
specified. In both cases, the PRP Entry shall have an offset of 0h. In a non -contiguous Submission
Queue, each PRP Entry in the PRP List shall have an offset of 0h. If there is a PRP Entry with a non -
zero offset, then the controller should return an error of PRP Offset Invalid.

Figure 478: Create I/O Submission Queue – Command Dword 10
Bits Description
31:16
Queue Size (QSIZE): This field specifies the size of the Submission Queue to be created. If the size is
0h or larger than the controller supports, the controller should return an error of Invalid Queue Size. Refer
to section 3.3.3.1. This is a 0’s based value.
15:00
Queue Identifier (QID): This field specifies  the identifier to assign to the Submission Queue to be
created. This identifier corresponds to the Submission Queue Tail Doorbell used for this command (i.e.,
the value y in SQyTDBL section of the NVMe over PCIe Transport Specification). This value shall not
exceed the value reported in the Number of Queues feature (refer to section 5.1.25.2.1) for I/O
Submission Queues. If the value specified is 0h, exceeds the Number of Queues reported, or
corresponds to an identifier already in use, the controller should return an error of Invalid Queue Identifier.
