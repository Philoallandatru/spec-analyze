---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 193
headings: []
---

# Source page 193

NVM Express® Base Specification, Revision 2.1

171
If a Format NVM command is in progress, then an Admin command not listed in Figure 142 that is submitted
for any namespace affected by that Format NVM command may be aborted. If aborted for that reason, then
a status code of Format in Progress should be returned.
If there are Admin commands not listed in Figure 142 being processed for a namespace, then a Format
NVM command which is submitted that affects that namespace may be aborted. If aborted for that reason,
then a status code of Command Sequence Error should be returned.
Figure 142: Sanitize Operations and Format NVM Command – Admin Commands Allowed
Admin Command Additional Restrictions for a
Format NVM Command
Additional Restrictions for a Sanitize
Operation
Abort
Asynchronous Event Request
Create I/O Completion Queue
Create I/O Submission Queue
Device Self–test Only Controller DST should be
allowed Prohibited
Delete I/O Completion Queue
Delete I/O Submission Queue
Get Features
Get Log Page
The log pages allowed for both Format NVM command and sanitize operations
are listed below.
Log Pages
Additional Restrictions for both
Format NVM command and
sanitize operations
Error Information Return 0h in the LBA field.
SMART / Health Information
Changed Attached Namespace List
Reservation Notification
Asymmetric Namespace Access
Sanitize Status
Vendor specific Prohibited for Sanitize
Persistent Event Log Prohibited for Sanitize
Boot Partition

Identify
Keep Alive
NVMe-MI Receive Allowed unless explicitly prohibited in the NVM Express Management Interface
Specification. NVMe-MI Send
Sanitize
Prohibited during sanitize operations if the
SANACT field is set to a value other than
101b (i.e., Exit Media Verification State).
Sanitize operations are described in
section 8.1.24.
Set Features Namespace Write Protection Config Feature is not allowed.
Opcode 7Fh
The Fabrics commands allowed are listed below. Refer to section 6.
Fabrics command Additional Restrictions for a Format NVM
Command and a Sanitize Operation
Property Set
Connect
Disconnect
Property Get
Authentication Send
Authentication Receive
Vendor Specific Commands are allowed that do not affect or
retrieve user data.

Vendor Specific Commands are allowed that do not affect or retrieve user data.
