---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 238
headings: ["5.1.12.1.7 Device Self-test (Log Page Identifier 06h)"]
---

# Source page 238

NVM Express® Base Specification, Revision 2.1

216
Figure 210: Commands Supported and Effects Data Structure
Bits Description
15:14
Command Submission and Execution Relaxations (CSER):  This field defines relaxed
command submission and execution recommendations for the associated command.
If this field is cleared to 00b, then the CSE field defines the command submission and execution
recommendations for the associated command and this field has no effect.
If this field is set to a non-zero value, then:
• if the host supports the value in this field, then the host should ignore the CSE field and
use the value in this field as the command submission and execution recommendation;
and
• if the host does not support the value in this field, then the host should ignore this field
and use the value in the CSE field as the command submission and execution
recommendation.
Value Definition
00b
No relaxed command submission and execution recommendation is defined.
The host should use the value in the CSE field as the command submission
and execution recommendation.
01b
The command associated with this structure should only be submitted when
there is no outstanding Admin command that affects any namespace and no
Admin command should be submitted that affects any namespace until this
command is complete.
10b to 11b Reserved

13:05 Reserved
04
Controller Capability Change (CCC): If this bit is set to ‘1’, then this command may change
controller capabilities. If this bit is cleared to ‘0’, then this command does not modify controller
capabilities. Controller capability changes include a firmware update that changes the capabilities
reported in the CAP property.
03
Namespace Inventory Change (NIC): If this bit is set to ‘1’, then this command may change the
number of namespaces or capabilities for multiple namespaces. If this bit is cleared to ‘0’, then
this command does not modify the number of namespaces or capabilities for multiple
namespaces. Namespace inventory changes include adding or removing namespaces.
02
Namespace Capability Change (NCC): If this bit is set to ‘1’, then this command may change
the capabilities of a single namespace.  If this bit is cleared to ‘0’, then this command does not
modify any namespace capabilities for the specified namespace. Namespace capability changes
include a logical format change.
01
Logical Block Content Change (LBCC): If this bit is set to ‘1’, then this command may modify
user data content in one or more namespaces that contain formatted storage. If this bit is cleared
to ‘0’, then this command does not modify user data content in any namespace. User data content
changes include a write to user data.
NOTE: This field applies to all user data content changes . The original name has been retained
for historical continuity.
00
Command Supported (CSUPP): If this bit is set to ‘1’, then this command is supported by the
controller. If this bit is cleared to ‘0’, then this command is not supported by the controller and all
other fields in this structure shall be cleared to 0h.
5.1.12.1.7 Device Self-test (Log Page Identifier 06h)
This log page is used to indicate:
a) the status of any device self -test operation in progress and the percentage complete of that
operation; and
b) the results of the last 20 device self-test operations.
The Self-test Result data structure (refer to Figure 212) contained in the Result Data Structure 1 field is
always the result of the most recently completed or aborted self-test operation. The next entry in the Device
Self-test Result list in the Device Self-test log page contains the results of the second most recent self-test
