---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 483
headings: []
---

# Source page 483

NVM Express® Base Specification, Revision 2.1

461
To abort an Admin command, the host uses the Abort command (refer to section 5.1.1). To abort a Cancel
command, the host may:
• use the Abort command;
• use a second Cancel command with a Cancel Action set to Single Command Cancel (i.e., 00b) that
specifies the CID of the Cancel command to abort; or
• follow the procedure described in section 3.3.1.3 to delete the I/O Submission Queue and recreate
the I/O Submission Queue.
The controller applies the specified Cancel Action (refer to Figure 557) to all outstanding commands that
have been fetched prior to the processing of the Cancel command. The controller may or may not apply
the specified Cancel Action to commands that are fetched:
• after the start of processing of the Cancel command; and
• before the completion of the Cancel command.
There is no requirement to apply the specified Cancel Action to commands that have not been fetched by
the controller.
If an abort action is performed on a command to abort, that action may be:
• an immediate abort (i.e., the abort action occurs prior to posting the completion queue entry for the
Cancel command); or
• a deferred abort (i.e., the abort action may or may not occur prior to posting the completion queue
entry for the Cancel command).
For each command on which an immediate abort is performed, the controller shall meet the immediate
abort requirements (refer to section 5.1.1.1) for that command before posting the completion queue entry
for the Cancel command.
For each command on which a deferred abort is performed, there are no requirements on the ordering of
posting of the completion queue entry for the Cancel command (refer to section 7.1.1).
For each command on which an abort action is performed (i.e., immediate abort or deferred abort) the
status code shall be set to Command Abort Requested in the completion queue entry for that command.
If the Cancel command is supported, then the Commands Supported and Effects log page shall be
supported.
The Cancel command uses the Command Dword 10 and Command Dword 11 fields. All other command
specific fields are reserved.
Figure 556: Cancel – Command Dword 10
Bits Description
31:16
Command Identifier (CID): This field specifies the command identifier of the command to be aborted
(i.e., the CDW0.CID field within the command to be aborted).
If the Action Code field is set to Single Command Cancel and this field is set to the CID for this Cancel
command, then the controller shall abort the command with a status code of Invalid Command ID.
If the Action Code field is set to Multiple Command Cancel and this field is not set to FFFFh, then the
controller shall abort the command with a status code of Invalid Field in Command.
15:00
Submission Queue Identifier (SQID): This field specifies the identifier of the Submission Queue that
the Cancel command is associated with.
If the SQID does not match the Submission Queue Identifier of the submission queue to which the Cancel
command is submitted, then the controller shall abort the command with a status code of Invalid Field in
Command.
