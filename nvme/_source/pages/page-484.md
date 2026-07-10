---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 484
headings: ["7.1.1 Command Completion"]
---

# Source page 484

NVM Express® Base Specification, Revision 2.1

462
Figure 557: Cancel – Command Dword 11
Bits Description
31:02 Reserved
01:00
Action Code (ACODE):
Value Cancel Action
00b
Single Command Cancel:  The hosts requests that the controller abort the
specific command submitted to the specified namespace with the specified CID.
If the NSID field is set to FFFFFFFFh then, the host requests that the controller
abort the specific command submitted to any NSID with the specified CID.
If the NSID is not set to FFFFFFFFh and the specified CID is not associated with
the specified NSID then, the controller shall abort the Cancel command with a
status code of Invalid Field in Command.
If the specified CID is not found, then the controller shall complete the Cancel
command with the Commands Eligible for Deferred Abort field cleared to 0h and
the Commands Aborted field cleared to 0h.
01b
Multiple Command Cancel:  The hosts requests that the controller abort all
commands, other than this Cancel command, on the I/O Submission Queue to
which the Cancel command was submitted that were submitted to the specified
NSID.
If the NSID is set to FFFFFFFFh then the host requests that the controller abort
all commands, other than this Cancel command, submitted to the I/O Submission
Queue to which the Cancel command was submitted for all namespaces.
All others Reserved

 Command Completion
Upon completion of the Cancel command, the controller posts a completion queue entry to the I/O
Completion Queue indicating the status for the Cancel command.
If the Cancel Action (refer to Figure 557) specified a Single Command Cancel Action and the Commands
Aborted field is cleared to 0h, then the host should examine the status in the completion queue entry of the
command to abort to determine whether the command was aborted or not (i.e., whether a d eferred abort
was performed or not).
If the Cancel Action specified a Multiple Command Cancel Action, then the host should examine the status
in the completion queue entry of each command to abort to determine whether the command was aborted
or not.
Cancel command specific status code values are defined in Figure 558.
Figure 558: Cancel – Command Specific Status Values
Value Description
84h Invalid Command ID: The specified CID matched the CID of this Cancel command.
Dword 0 of the completion queue entry contains information about the number of commands that were
aborted by this command. Dword 0 of the completion queue entry is defined in Figure 559.
