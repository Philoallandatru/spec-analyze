---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 284
headings: ["5.1.12.1.20 Command and Feature Lockdown (Log Page Identifier 14h)"]
---

# Source page 284

NVM Express® Base Specification, Revision 2.1

262
Figure 264: NVMe-MI Commands Supported and Effects Data Structure
Bits Description
31:20
Command Scope (CSP): This field defines the scope for the associated NVMe -MI Send command
that specifies the Management Interface command opcode for this data structure. If the value of this
field is 0h, then no scope is reported. If this field is non-zero, then only one bit shall be set to ‘1’.
Bits Description
11:6 Reserved
5
NVM Subsystem Scope  (NSSCPE): If this bit is set to ‘1’, then the command
performs actions that may impact the whole NVM subsystem. If this bit is cleared to
‘0’ and the CSP field is non -zero, then the command performs actions that do not
impact the whole NVM subsystem.
4
Domain Scope  (DSCPE): If this bit is set to ‘1’, then the command has Domain
scope. If this bit is cleared to ‘0’ and the CSP field is non -zero, then the command
does not have Domain scope.
3
Endurance Group Scope (EGSCPE): If this bit is set to ‘1’, then the command has
Endurance Group scope. If this bit is cleared to ‘0’ and the CSP field is non -zero,
then the command does not have Endurance Group scope.
2
NVM Set Scope (NSSCPE): If this bit is set to ‘1’, then the command has NVM Set
scope. If this bit is cleared to ‘0’ and the CSP field is non -zero, then the command
does not have NVM Set scope.
1
Controller Scope (CSCPE): If this bit is set to ‘1’, then the command has controller
scope. If this bit is cleared to ‘0’ and the CSP field is non -zero, then the command
does not have controller scope.
0
Namespace Scope  (NSCPE): If this bit is set to ‘1’, then the command has
namespace scope. If this bit is cleared to ‘0’ and the CSP field is non-zero, then the
command does not have namespace scope.

19:05 Reserved
04
Controller Capability Change (CCC): If this bit is set to ‘1’, then this command may change controller
capabilities. If this bit is cleared to ‘0’, then this command does not modify controller capabilities.
Controller capability changes include a firmware update that changes the capabilities reported in the
CAP property.
03
Namespace Inventory Change (NIC): If this bit is set to ‘1’, then this command may change the
number of namespaces or capabilities for multiple namespaces. If this bit is cleared to ‘0’, then this
command does not modify the number of namespaces or capabilities for multiple namespaces.
Namespace inventory changes include adding or removing namespaces.
02
Namespace Capability Change (NCC): If this bit is set to ‘1’, then this command may change the
capabilities of a single namespace. If this bit is cleared to ‘0’, then this command does not modify any
namespace capabilities for the specified namespace. Namespace capability changes include a logical
format change.
01
User Data Content Change (UDCC): If this bit is set to ‘1’, then this command may modify user data
content in one or more namespaces. If this bit is cleared to ‘0’, then this command does not modify
user data content in any namespace. User data content changes include a write operation.
00
Command Supported (CSUPP): If this bit is set to ‘1’, then this command is supported by the
controller. If this bit is cleared to ‘0’, then this command is not supported by the controller and all other
fields in this structure shall be cleared to 0h.
5.1.12.1.20 Command and Feature Lockdown (Log Page Identifier 14h)
This log page is used to indicate which commands and Set Features Feature Identifiers are supported to
be prohibited from execution using the Command and Feature Lockdown capability (refer to section 8.1.5)
and which commands are currently prohibited if received on an Admin Submission Queue or received out -
of-band on a Management Endpoint (refer to the NVM Express Management Interface Specification ). This
log page uses the Log Specific Parameter field in Command Dword 10 (refer to Figure 197) as defined in
Figure 265. This log page may use the UUID Index field in the Get Log Page command to specify the scope
and content of the list returned in the Command and Feature Identifier List field of this log page. The UUID
Index field may be used if the Scope field is set to 2h, allowing re turning of vendor specific Set Features
Feature Identifier lockdown information.
