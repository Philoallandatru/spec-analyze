---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 166
headings: ["4.2.3.2 Command Specific Status Definition"]
---

# Source page 166

NVM Express® Base Specification, Revision 2.1

144
Figure 102: Status Code – Generic Command Status Values
Value Description I/O Command
Set(s)1
29h FDP Disabled: The command is not allowed when Flexible Data Placement is
disabled.
2Ah
Invalid Placement Handle List: The Placement Handle List is invalid due to:
• a Reclaim Unit Handle Identifier that is:
o valid but restricted to be used by the command; or
o invalid;
or
• the Placement Handle List number of entries exceeded the maximum
number allowed.

2Bh to 7Fh Reserved
80h LBA Out of Range:  See the applicable I/O Command Set specification for the
description. NVM, ZNS
81h Capacity Exceeded: The command attempted an operation that exceeds the capacity
of the namespace.
82h
Namespace Not Ready: The namespace is not ready to be accessed as a result of a
condition other than a condition that is reported as an Asymmetric Namespace Access
condition. The Do Not Retry bit indicates whether re -issuing the command at a later
time may succeed.

83h Reservation Conflict: The command was aborted due to a conflict with a reservation
held on the accessed namespace. Refer to section 8.1.22.
84h
Format In Progress: A Format NVM command is in progress on the namespace. The
Do Not Retry bit shall be cleared to ‘0’ to indicate that the command may succeed if
resubmitted.
NVM, ZNS
85h Invalid Value Size:  See the applicable I/O Command Set specification for the
description.  KV
86h Invalid Key Size:  See the applicable I/O Command Set specification for the
description.  KV
87h KV Key Does Not Exist:  See the applicable I/O Command Set specification for the
description.  KV
88h Unrecovered Error:  See the applicable I/O Command Set specification for the
description.  KV
89h Key Exists: See the applicable I/O Command Set specification for the description.  KV
90h to BFh Reserved
C0h to FFh Vendor Specific
Key:
NVM – NVM Command Set
ZNS – Zoned Namespace Command Set
KV – Key Value Command Set
Notes:
1. This column is blank unless the value is I/O Command Set specific
 Command Specific Status Definition
Completion queue entries with a Status Code Type (SCT) of Command Specific Errors indicate an error
that is specific to a particular command opcode. Status codes of 00h to 7Fh are for Admin command errors.
Status codes of 80h to BFh are specific to the selected I/O command sets.
Figure 103: Status Code – Command Specific Status Values
Value Description Commands Affected
00h Completion Queue Invalid  Create I/O Submission Queue
