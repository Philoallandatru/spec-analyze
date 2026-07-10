---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 540
headings: []
---

# Source page 540

NVM Express® Base Specification, Revision 2.1

518
Figure 616: Flexible Data Placement Model

A write command that places user data into a Reclaim Unit causes reduction of the remaining capacity of
that Reclaim Unit. If that Reclaim Unit is written to capacity, then the controller modifies the Reclaim Unit
Handle referencing that Reclaim Unit to reference a different Reclaim Unit within the same Reclaim Group.
A controller may modify a Reclaim Unit Handle to reference a different Reclaim Unit as part of performing
a sanitize operation (refer to section 8.1.24).
A Reclaim Unit shall only be referenced by one Reclaim Unit Handle. Therefore, a write command that
utilizes a Reclaim Unit Handle results in the user data from that write command being initially isolated from
the user data from any other write command tha t utilizes a different Reclaim Unit Handle. Each Reclaim
Unit Handle has a type as defined in Figure 281 that identifies the isolation requirements of the user data
moved by the controller to a different Reclaim Unit due to vendor specific controller operations (e.g.,
garbage collection).
A controller is allowed to move user data that was written by the host that utilized a Reclaim Unit Handle
with an Initially Isolated type ( Figure 281) to a different Reclaim Unit within the same Reclaim Group. In
addition, the controller is allowed to move user data written by the host that utilized different Reclaim Unit
Handles with an Initially Isolated type to that different Reclaim Unit within the  same Reclaim Group as
shown in Figure 617.
A controller is allowed to move user data that was written by the host that utilized a Reclaim Unit Handle
with a Persistently Isolated type ( Figure 281) to a different Reclaim Unit within the same Reclaim Group.
However, the controller shall only move user data written by the host that utilized the same Reclaim Unit
Handles with a Persistently Isolated type to the different Reclaim Unit within the same R eclaim Group as
shown in Figure 618.
Endurance Group
Reclaim
Group 1
Reclaim
Unit
Reclaim
Unit
...
Reclaim
Group 0
Reclaim
Unit
Reclaim
Unit
...
Reclaim
Group 2
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
...
Reclaim Group
NRG-1
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
...
...
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
Reclaim Unit Handle 0
...
I/O write command
NSID: Namespace A
DSPEC (i.e, Placement Identifer )
Reclaim Group
Identifier
Placement
Handle
Namespace A
Placement
Handle
10
NRUH-11
0X
...
Reclaim Unit Handle 1Reclaim Unit
Handle Identifier
Reclaim Unit Handle NRUH-1
The number of entries is defined by the
Namespace Management command as
defined by the appropriate I /O
Command Set specification .
