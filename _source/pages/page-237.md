---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 237
headings: []
---

# Source page 237

NVM Express® Base Specification, Revision 2.1

215
Figure 210: Commands Supported and Effects Data Structure
Bits Description
31:20
Command Scope (CSP): This field defines the scope for the associated command. If the value
of this field is 0h then no scope is reported. A command may have multiple scopes depending on
the effects of the command based on the parameters passed to that command. For a command
that supports multiple scopes, multiple bits may set to ‘1’.
Bits Description
11:6 Reserved
5
NVM Subsystem Scope  (NSSCPE): If this bit is set to ‘1’, then the command
performs actions that may impact the whole NVM subsystem. If this bit is cleared to
‘0’ and the CSP field is non-zero, then the command performs actions that does not
impact the whole NVM subsystem.
4
Domain Scope (DSCPE): If this bit is set to ‘1’, then the command performs actions
that may impact a single Domain. If this bit is cleared to ‘0’ and the CSP field is non-
zero, then the command performs actions that does not impact a single Domain.
3
Endurance Group Scope  (EGSCPE): If this bit is set to ‘1’, then the command
performs actions that may impact Endurance Groups. If  this bit is cleared to ‘0’ and
the CSP field is non-zero, then the command performs actions that does not impact
a single Endurance Group.
2
NVM Set Scope  (NSSCPE): If this bit is set to ‘1’, then the command performs
actions that may impact NVM Sets. If this bit is cleared to ‘0’ and the CSP field is
non-zero, then the command performs actions that do not impact NVM Sets.
1
Controller Scope  (CSCPE): If this bit is set to ‘1’, then the command performs
actions that may impact controller s. If this bit is cleared to ‘0’ and the CSP field is
non-zero, then the command performs actions that do not impact controllers.
0
Namespace Scope (NSCPE): If this bit is set to ‘1’, then the command performs
actions that may impact namespaces. If this bit is cleared to ‘0’ and the CSP field is
non-zero, then the command performs actions that do not impact namespaces.

19
UUID Selection Supported (USS): If this bit is set to ‘1’, then the controller supports selection
of a UUID by this command (refer to section 8.1.28). If this bit is cleared to ‘0’, then the controller
does not support selection of a UUID by this command.
18:16
Command Submission and Execution (CSE): This field defines the command submission and
execution recommendations for the associated command.
If the CSER field contains a non-zero value and the host supports the value in that field, then this
field should be ignored by the host.  If the CSER field is set to 01b, then this field shall be set to
001b.
Value Definition
000b No command submission or execution restriction
001b
The command associated with this structure should only be submitted
when there is no other outstanding command affecting the same
namespace and another command should not be submitted that affects
the same namespace until this command is complete.
010b
The command associated with this structure should only be submitted
when there is no other outstanding command that affects any
namespace and another command should not be submitted that affects
any namespace until this command is complete.
011b to 111b Reserved
