---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 163
headings: ["4.2.3.1 Generic Command Status Definition"]
---

# Source page 163

NVM Express® Base Specification, Revision 2.1

141
Figure 101: Status Code – Status Code Type Values
Value Description Reference
7h Vendor Specific
The Status Code (SC) field in the completion queue entry indicates more detailed status information about
the completion being reported.
Each Status Code set of values is split into three ranges:
• 00h to 7Fh: Applicable to Admin Command Set, or across multiple command sets;
• 80h to BFh: I/O Command Set specific status codes; and
• C0h to FFh: Vendor Specific status codes.
Unless otherwise specified, if multiple status codes apply, then the controller selects the status code that is
returned.
 Generic Command Status Definition
Completion queue entries with a Status Code Type (SCT) of Generic Command Status indicate a status
value associated with the command that is generic across many different types of commands.
Figure 102: Status Code – Generic Command Status Values
Value Description I/O Command
Set(s)1
00h Successful Completion: The command completed without error.
01h Invalid Command Opcode: A reserved coded value or an unsupported value in the
command opcode field.
02h
Invalid Field in Command:  A reserved coded value or an unsupported value in a
defined field (other than the opcode field). This status code should be used unless
another status code is explicitly specified for a particular condition. The field may be in
the command parameters as part of the submission queue entry or in data structures
pointed to by the command parameters.

03h Command ID Conflict: The command identifier is already in use. Note: It is
implementation specific how many commands are searched for a conflict.
04h Data Transfer Error: Transferring the data or metadata associated with a command
had an error.
05h Commands Aborted due to Power Loss Notification:  Indicates that the command
was aborted due to a power loss notification.
06h
Internal Error: The command was not completed successfully due to an internal error.
Details on the internal device error should be reported as an asynchronous event.
Refer to Figure 149 for Internal Error Asynchronous Event Information.

07h
Command Abort Requested: The command was aborted due to:
• an Abort command being received that specified this command (refer to
section 5.1); or
• a Cancel command being received that specified this command (refer to
section 7.1).

08h
Command Aborted due to SQ Deletion: The command was aborted due to a Delete
I/O Submission Queue request received for the Submission Queue to which the
command was submitted.

09h Command Aborted due to Failed Fused Command:  The command was aborted
due to the other command in a fused operation failing.
0Ah
Command Aborted due to Missing Fused Command:  The fused command was
aborted due to the adjacent submission queue entry not containing a fused command
that is the other command in a supported fused operation (refer to section 3.4.2).

0Bh Invalid Namespace or Format: The namespace or the format of that namespace is
invalid.
