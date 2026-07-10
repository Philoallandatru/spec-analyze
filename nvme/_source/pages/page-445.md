---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 445
headings: ["5.2.4.1 Command Completion", "5.2.5 Doorbell Buffer Config command"]
---

# Source page 445

NVM Express® Base Specification, Revision 2.1

423
completion of the Delete I/O Submission Queue command, the controller shall not post completion status
for any I/O commands that were submitted to the deleted I/O Submission Queue. The successful
completion of the Delete I/O Submission Queue command indica tes an implicit completion status of
Command Aborted due to SQ Deletion for any previously submitted I/O commands that did not have a
completion queue entry posted by the controller.
Note: It is not possible to delete the Admin Submission Queue.
Figure 484: Delete I/O Submission Queue – Command Dword 10
Bits Description
31:16 Reserved
15:00 Queue Identifier (QID): This field indicates the identifier of the Submission Queue to be deleted. The
value of 0h (Admin Submission Queue) shall not be specified.
 Command Completion
After all commands submitted to the indicated I/O Submission Queue are either completed or aborted, a
completion queue entry is posted to the Admin Completion Queue when the queue has been deleted.
Delete I/O Submission Queue command specific status values are defined in Figure 485.
Figure 485: Delete I/O Submission Queue – Command Specific Status Values
Value Definition
1h Invalid Queue Identifier: The Queue Identifier specified in the command is invalid. This error is also
indicated if the Admin Submission Queue identifier is specified.
 Doorbell Buffer Config command
The Doorbell Buffer Config command is used to provide two separate memory buffers that mirror the
controller's doorbell properties defined in section 3.1.4. This command is intended for emulated controllers
and is not typically supported by a physical NVMe controller. The two buffers are known as “Shadow
Doorbell” and “EventIdx”, respectively. Refer to “ Updating Controller Doorbell Properties using a Shadow
Doorbell Buffer” in section B.5.2 for an example of how these buffers may be used.
The Doorbell Buffer Config command uses the PRP Entry 1 and PRP Entry 2 fields. All other command
specific fields are reserved. The command is not namespace specific, does not support metadata, and does
not support SGLs. The settings are not retained across a Controller Level Reset.
Each buffer supplied with the Doorbell Buffer Config command shall be a single physical memory page as
defined by the CC.MPS field. The controller shall ensure that the following condition is satisfied:
 (4 << CAP.DSTRD) * (max(NSQA, NCQA)+1) <= (2^(12+CC.MPS))
Figure 486: Doorbell Buffer Config – Shadow Doorbell and EventIdx
Start
(Offset in Buffer)1, 2
End
(Offset in Buffer)1, 2 Description 2
00h 03h Submission Queue 0 Tail Doorbell or EventIdx (Admin)
00h + (1 *
(4 << CAP.DSTRD))
03h + (1 *
(4 << CAP.DSTRD)) Completion Queue 0 Head Doorbell or EventIdx (Admin)
00h + (2 *
(4 << CAP.DSTRD))
03h + (2 *
(4 << CAP.DSTRD)) Submission Queue 1 Tail Doorbell or EventIdx
00h + (3 *
(4 << CAP.DSTRD))
03h + (3 *
(4 << CAP.DSTRD)) Completion Queue 1 Head Doorbell or EventIdx
… … …
00h + (2y *
(4 << CAP.DSTRD))
03h + (2y *
(4 << CAP.DSTRD)) Submission Queue y Tail Doorbell or EventIdx
00h + ((2y + 1) *
(4 << CAP.DSTRD))
03h + ((2y + 1) *
(4 << CAP.DSTRD)) Completion Queue y Head Doorbell or EventIdx
Notes:
