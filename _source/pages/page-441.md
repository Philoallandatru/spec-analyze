---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 441
headings: ["5.2.1 Create I/O Completion Queue command"]
---

# Source page 441

NVM Express® Base Specification, Revision 2.1

419
 Create I/O Completion Queue command
The Create I/O Completion Queue command is used to create all I/O Completion Queues with the exception
of the Admin Completion Queue. The Admin Completion Queue is created by specifying its base address
in the ACQ property. If a PRP List is provided to describe the CQ, then the PRP List shall be maintained by
host software at the same location in host physical memory and the values in the PRP List shall not be
modified until the corresponding Delete I/O Completion Queue command for this CQ is completed
successfully or the controller is reset. If the PRP List values are modified, the behavior is undefined.
The Create I/O Completion Queue command uses the PRP Entry 1, Command Dword 10, and Command
Dword 11 fields. All other command specific fields are reserved.
Figure 473: Create I/O Completion Queue – PRP Entry 1
Bits Description
63:00
PRP Entry 1 (PRP1): If CDW11.PC is set to ‘1’, then this field specifies a 64 -bit base memory address
pointer of the Completion Queue that is physically contiguous. The address pointer is memory page
aligned (based on the value in CC.MPS) unless otherwise specified. If CDW11.PC is cleared to ‘0’, then
this field specifies a PRP List pointer that describes the list of pages that constitute the Co mpletion
Queue. The list of pages is memory page aligned (based on the value in CC.MPS) unless otherwise
specified. In both cases the PRP Entry shall have an offset of 0h. In a non-contiguous Completion Queue,
each PRP Entry in the PRP List shall have an offset of 0h. If there is a PRP Entry with a non-zero offset,
then the controller should return an error of PRP Offset Invalid.

Figure 474: Create I/O Completion Queue – Command Dword 10
Bits Description
31:16
Queue Size (QSIZE): This field indicates the size of the Completion Queue to be created. If the size is
0h or larger than the controller supports, the controller should return an error of Invalid Queue Size. Refer
to section 3.3.3.1. This is a 0’s based value.
15:00
Queue Identifier (QID): This field indicates the identifier to assign to the Completion Queue to be
created. This identifier corresponds to the Completion Queue Head Doorbell used for this command (i.e.,
the value y in the CQyHDBL section of the NVMe over PCIe Transport Specification). This value shall
not exceed the value reported in the Number of Queues feature (refer to section 5.1.25.2.1) for I/O
Completion Queues. If the value specified is 0h, exceeds the Number of Queues reported, or
corresponds to an identifier already in use, the controller should return an error of Invalid Queue Identifier.

Figure 475: Create I/O Completion Queue – Command Dword 11
Bits Description
31:16
Interrupt Vector (IV): This field’s value is transport specific and is described in the applicable NVMe
Transport binding specification if required. If not defined by the transport, then this field shall be cleared
to 0h.
15:02 Reserved
01 Interrupts Enabled (IEN): If this bit is set to ‘1’, then interrupts are enabled for this Completion Queue.
If this bit is cleared to ‘0’, then interrupts are disabled for this Completion Queue.
00
Physically Contiguous (PC): If this bit is set to ‘1’, then the Completion Queue is physically contiguous
and PRP Entry 1 (PRP1) is the address of a contiguous physical buffer. If this bit is cleared to ‘0’, then
the Completion Queue is not physically contiguous and PRP Entry 1 (PRP1) is a PRP List pointer. If this
bit is cleared to ‘0’ and CAP.CQR is set to ‘1’, then the controller shall abort the command with a status
code of Invalid Field in Command.
If the:
• queue is located in the Controller Memory Buffer;
• PC is cleared to ‘0’; and
• CMBLOC.CQPDS is cleared to ‘0’,
then the controller shall abort the command with a status code of Invalid Use of Controller Memory Buffer.
