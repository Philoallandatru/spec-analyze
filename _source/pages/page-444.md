---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 444
headings: ["5.2.3 Delete I/O Completion Queue command", "5.2.3.1 Command Completion", "5.2.4 Delete I/O Submission Queue command"]
---

# Source page 444

NVM Express® Base Specification, Revision 2.1

422
Figure 481: Create I/O Submission Queue – Command Specific Status Values
Value Definition
0h Completion Queue Invalid:  The Completion Queue identifier specified in the command has not been
created.
1h
Invalid Queue Identifier: The creation of the I/O Submission Queue failed due an invalid queue identifier
specified as part of the command. An invalid queue identifier is one that identifies the Admin Queue (i.e., 0h),
is outside the range supported by the controller, or is a Submission Queue Identifier that is already in use.
2h
Invalid Queue Size: The host attempted to create an I/O Submission Queue:
• with an invalid number of entries (e.g., a value of 0h or a value which exceeds the maximum
supported by the controller, specified in CAP.MQES); or
• before initializing the CC.IOSQES field.
 Delete I/O Completion Queue command
The Delete I/O Completion Queue command is used to delete an I/O Completion Queue. The Delete I/O
Completion Queue command uses the Command Dword 10 field. All other command specific fields are
reserved. After this command has completed, the PRP List that describes the Completion Queue may be
deallocated by host software.
Host software shall ensure that any associated I/O Submission Queue is deleted prior to deleting a
Completion Queue. If there are any associated I/O Submission Queues present, then the Delete I/O
Completion Queue command shall abort with a status code of Invalid Queue Deletion.
Note: It is not possible to delete the Admin Completion Queue.
Figure 482: Delete I/O Completion Queue – Command Dword 10
Bits Description
31:16 Reserved
15:00 Queue Identifier (QID): This field indicates the identifier of the Completion Queue to be deleted. The
value of 0h (Admin Completion Queue) shall not be specified.
 Command Completion
Upon completion of the Delete I/O command, the controller posts a completion queue entry to the Admin
Completion Queue. Delete I/O Completion Queue command specific status values are defined in Figure
483.
Figure 483: Delete I/O Completion Queue – Command Specific Status Values
Value Definition
01h Invalid Queue Identifier:  The Queue Identifier specified in the command is invalid. This error is also
indicated if the Admin Completion Queue identifier is specified.
0Ch
Invalid Queue Deletion: This error indicates that it is invalid to delete the I/O Completion Queue specified.
The typical reason for this error condition is that there is an associated I/O Submission Queue that has not
been deleted.
 Delete I/O Submission Queue command
The Delete I/O Submission Queue command is used to delete an I/O Submission Queue. The Delete I/O
Submission Queue command uses the Command Dword 10 field. All other command specific fields are
reserved. After this command has completed, the PRP List that desc ribes the Submission Queue may be
deallocated by host software.
Upon successful completion of the Delete I/O Submission Queue command, all I/O commands previously
submitted to the indicated Submission Queue shall be either explicitly completed or implicitly completed.
Prior to returning a completion queue entry for the  Delete I/O Submission Queue command, other
commands previously submitted to the I/O Submission Queue to be deleted may be completed with
appropriate status (e.g., Successful Completion, Command Aborted due to SQ Deletion). After successful
