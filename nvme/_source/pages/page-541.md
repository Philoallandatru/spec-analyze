---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 541
headings: ["8.1.10.2 Enabling Flexible Data Placement (Informative)"]
---

# Source page 541

NVM Express® Base Specification, Revision 2.1

519
Figure 617: Initially Isolated Reclaim Unit Handles

Figure 618: Persistently Isolated Reclaim Unit Handle

The result of a Namespace Management command that creates a namespace within the Endurance Group
is that at least one Placement Handle is defined for that namespace (refer to the Namespace Management
section in applicable I/O Command Set specification).
For any namespace created in an Endurance Group that has Flexible Data Placement enabled, the host
may issue an I/O Management Receive command to obtain a list of Placement Handles and the associated
Reclaim Unit Handles that are used by the Data Placement Directive in write commands.
If the Flexible Data Placement capability is supported, then the controller shall support the following:
a) the Feature Identifiers Supported and Effects log page (refer to section 5.1.12.1.18);
b) the I/O Management Receive command (refer to section 7.3) and the Reclaim Unit Handle Status
Management Operation in that command (refer to Figure 561);
c) the I/O Management Send command (refer to section 7.4) and the Placement Identifier Update
Management Operation in that command (refer to Figure 565);
d) the Flexible Data Placement feature (refer to section 5.1.25.1.20);
e) the FDP Configurations log page (refer to section 5.1.12.1.28);
f) the Reclaim Unit Handle Usage log page (refer to section 5.1.12.1.29);
g) the FDP Statistics log page (refer to section 5.1.12.1.30);
h) the Flexible Data Placement Events feature (refer to section 5.1.25.1.21);
i) the FDP Events log page (refer to section 5.1.12.1.31); and
j) the Data Placement Directive (refer to section 8.1.8.4).
 Enabling Flexible Data Placement (Informative)
The host prepares an Endurance Group for operation in Flexible Data Placement using the following
process:
