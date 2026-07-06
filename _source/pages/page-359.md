---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 359
headings: []
---

# Source page 359

NVM Express® Base Specification, Revision 2.1

337
Figure 319: Identify – I/O Command Set Independent Identify Namespace Data Structure
Bytes O/M1 Description Reported2
03 O
Format Progress Indicator (FPI): If a format operation is in progress, this field
indicates the percentage of the namespace that remains to be formatted.
Bits Description
7
Format Progress Indicator Support (FPIS): If this bit is set to ‘1’, then
the namespace supports the Format Progress Indicator defined by the
RFNVM field. If this bit is cleared to ‘0’, then the namespace does not
support the Format Progress Indicator.
6:0
Remaining Format NVM (RFNVM): This field indicates the percentage
of the Format NVM command that remains to be completed (e.g., a value
of 25 indicates that 75% of the Format NVM command has been
completed and 25% remains to be completed). If the FPIS bit is set to ‘1’,
then a value of 0h indicates that the namespace is formatted with the
format specified by Identify Namespace data structures (refer to section
1.5.49) and there is no Format NVM command in progress.
If the FPIS bit is cleared to’0’, then this field shall be cleared to 0h.

No
07:04 O
ANA Group Identifier (ANAGRPID): For NSID other than FFFFFFFFh, this field
indicates the ANA Group Identifier of the ANA group (refer to section 8.1.1.2) of
which the namespace is a member. Each namespace that is attached to a
controller that supports Asymmetric Namespace Access Reporting (refer to the
CMIC field) shall report a valid ANAGRPID. If the controller does not support
Asymmetric Namespace Access Reporting, then this field shall be cleared to 0h.
If the value in this field changes and Asymmetric Namespace Access Change
Notices are supported and enabled, then the controller shall issue an Asymmetric
Namespace Access Change Notice.
No
08 O
Namespace Attributes (NSATTR):  This field specifies attributes of the
namespace.
Bits Description
7:1 Reserved
0
Currently Write Protected (CWP) : If this bit is set to ‘1’, then the
namespace is currently write protected due to any condition (e.g.,
namespace write protection set for the namespace, media errors) and all
write access to the namespace shall fail. If this bit is cleared to ‘0’, then
the namespace is not currently write protected.

No
09  Reserved
11:10 O
NVM Set Identifier (NVMSETID):  For NSID other than FFFFFFFFh, this field
indicates the NVM Set with which this namespace is associated. If NVM Sets are
not supported by the controller, then this field shall be cleared to 0h.
For namespaces that do not contain formatted storage this field shall be cleared to
0h.
No
13:12 O
Endurance Group Identifier (ENDGID):  For NSID other than FFFFFFFFh, this
field indicates the Endurance Group with which this namespace is associated. If
Endurance Groups are not supported by the controller, then this field shall be
cleared to 0h.
For namespaces that do not contain formatted storage this field shall be cleared to
0h.
No
