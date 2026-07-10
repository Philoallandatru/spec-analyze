---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 325
headings: []
---

# Source page 325

NVM Express® Base Specification, Revision 2.1

303
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
13
Delete Endurance Group (DEG): If this bit is set to ‘1’, then the controller
supports the Delete Endurance Group operation (refer to section 8.1.4.3).
If this bit is cleared to ‘0’, then the controller does not support the Delete
Endurance Group operation.
12
Variable Capacity Management  (VCM): If this bit is set to ‘1’, then the
controller supports Variable Capacity Management (refer to section
8.1.4.3). If this bit is cleared to ‘0’, then the controller does not support
Variable Capacity Management.
11
Fixed Capacity Management  (FCM): If this bit is set to ‘1’, then the
controller supports Fixed Capacity Management (refer to section 8.1.4.2).
If this bit is cleared to ‘0’, then the controller does not support Fixed
Capacity Management.
If the Flexible Data Placement capability is supported (i.e., the FDPS bit is
set to ‘1’), then this bit shall be cleared to ‘0’.
10
Multi-Domain Subsystem (MDS):  If this bit is set to ‘1’, then the NVM
subsystem supports the multiple domains (refer to section 3.2.5). If this bit
is cleared to ‘0’, then the NVM subsystem does not support the reporting
of multiple domains and the NVM subsystem consists of a single domain.
9
UUID List  (ULIST): If this bit is set to ‘1’, then the controller supports
reporting of a UUID List (refer to Figure 320). If this bit is cleared to ‘0’, then
the controller does not support reporting of a UUID List (refer to section
8.1.28).
8
SQ Associations (SQA): If this bit is set to ‘1’, then the controller supports
SQ Associations (refer to section 8.1.25). If this bit is cleared to ‘0’, then
the controller does not support SQ Associations.
7
Namespace Granularity (NG): If this bit is set to ‘1’, then the controller
supports reporting of Namespace Granularity (refer to section 5.1.13.2.13).
If this bit is cleared to ‘0’, then the controller does not support reporting of
Namespace Granularity. If the Namespace Management capability (refer
to section 8.1.15) is not supported, then this bit shall be cleared to ‘0’.
6
Traffic Based Keep Alive Support (TBKAS):  If this bit is set to '1‘, then
the controller uses Traffic Based Keep Alive (refer to section 3.9.4.1). If this
bit is cleared to '0‘, then the controller does not use Traffic Based Keep
Alive.
5
Predictable Latency Mode (PLM): If this bit is set to ‘1’, then the controller
supports Predictable Latency Mode (refer to section 8.1.18). If this bit is
cleared to ‘0’, then the controller does not support Predictable Latency
Mode.
4
Endurance Groups  (EGS): If this bit is set to ‘1’, then the controller
supports Endurance Groups (refer to section 3.2.3). If this bit is cleared to
‘0’, then the controller does not support Endurance Groups.
3
Read Recovery Levels (RRLVLS): If this bit is set to ‘1’, then the controller
supports Read Recovery Levels (refer to section 8.1.20). If this bit is
cleared to ‘0’, then the controller does not support Read Recovery Levels.
2
NVM Sets (NSETS): If this bit is set to ‘1’, then the controller supports NVM
Sets (refer to section 3.2.2). If this bit is cleared to ‘0’, then the controller
does not support NVM Sets.
