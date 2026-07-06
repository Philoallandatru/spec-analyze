---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 282
headings: []
---

# Source page 282

NVM Express® Base Specification, Revision 2.1

260
Figure 261: Feature Identifiers Effects Log Page
Bytes Description
07:04 Feature Identifier Supported 1 (FIS1): Contains the FID Supported and Effects data structure (refer
to Figure 262) for FID 1h.
… …
1019:1016 Feature Identifier Supported 254 (FIS254): Contains the FID Supported and Effects data structure
(refer to Figure 262) for FID FEh.
1023:1020 Feature Identifier Supported 255 (FIS255): Contains the FID Supported and Effects data structure
(refer to Figure 262) for FID FFh.
The FID Supported and Effects data structure describes the effect of a Set Features command for the FID,
including any optional features of the FID.
Figure 262: FID Supported and Effects Data Structure
Bits Description
31:20
FID Scope (FSP): This field defines the scope for the associated feature identifier. If the value of this
field is 0h, then no scope is reported. If this field is non-zero, then only one bit shall be set to ‘1’.
Bits Description
11:7 Reserved
6
Controller Data Queue (CDQSCP) : If set to ‘1’, then modifying the attributes of the
feature identified by this FID may impact a Controller Data Queue (refer to 8.1.6). If
cleared to ‘0’ and the FSP field is non -zero, then modifying the attributes of the feature
identified by this FID does not impact a Controller Data Queue.
5
NVM Subsystem Scope (NSSCPE): If this bit is set to ‘1’, then modifying the attributes
of the feature identified by this FID  may impact the whole NVM subsystem. If this bit is
cleared to ‘0’ and the FSP field is non -zero, then modifying the attributes of the feature
identified by this FID does not impact the whole NVM subsystem.
4
Domain Scope  (DSCPE): If this bit is set to ‘1’, then modifying the attributes of the
feature identified by this FID may impact a single Domain. If this bit is cleared to ‘0’ and
the FSP field is non -zero, then modifying the attributes of the feature identified by this
FID does not impact a single Domain.
3
Endurance Group Scope (EGSCPE): If this bit is set to ‘1’, then modifying the attributes
of the feature identified by this FID  may impact Endurance Groups. If this bit is cleared
to ‘0’ and the FSP field is non-zero, then modifying the attributes of the feature identified
by this FID does not impact Endurance Groups.
2
NVM Set Scope (NSSCPE): If this bit is set to ‘1’, then modifying the attributes of the
feature identified by this FID  may impact NVM Sets. If this bit is cleared to ‘0’ and the
FSP field is non -zero, then modifying the attributes of the feature identified by this FID
does not impact NVM Sets.
1
Controller Scope (CSCPE): If this bit is set to ‘1’, then modifying the attributes of the
feature identified by this FID may impact the controller. If this bit is cleared to ‘0’ and the
FSP field is non -zero, then the feature identified by this FID does not have controller
scope.
0
Namespace Scope (NSCPE): If this bit is set to ‘1’, then modifying the attributes of the
feature identified by this FID may impact namespaces. If this bit is cleared to ‘0’ and the
FSP field is non -zero, then modifying the attributes of the feature identified by this FID
does not impact namespaces.

19
UUID Selection Supported (USS): If this bit is this bit is set to ‘1’, then the controller supports the
selection of a UUID (refer to section 8.1.28) by a Get Features command or a Set Features command
using this FID.
If this bit is cleared to ‘0’, then the controller does not support the selection of a UUID by a Get
Features command or a Set Features command using this FID.
18:05 Reserved
