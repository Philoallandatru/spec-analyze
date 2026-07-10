---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 500
headings: ["8.1.1.3 Asymmetric Namespace Access states", "8.1.1.4 ANA Optimized state", "8.1.1.5 ANA Non-Optimized state"]
---

# Source page 500

NVM Express® Base Specification, Revision 2.1

478
Figure 587: Multiple Namespace groups
NSID 1 NSID 2
NVMe Controller 1
NS
A
NS
B
NVMe Controller 2
Port x Port yNVM
Subsystem
Port z
NVMe Controller 3
NSID 2
NS
D
NSID 4 NSID 4NSID 4
NS
C
NSID 3NSID 2 NSID 3
Media Access
Boundary
ANA Group
ANA Group
ANA Group

 Asymmetric Namespace Access states
The Asymmetric Namespace Access State indicates information about the characteristics of the
relationship between a controller and an ANA Group. The following asymmetric namespace access states
are defined:
• ANA Optimized (refer to section 8.1.1.4);
• ANA Non-Optimized (refer to section 8.1.1.5);
• ANA Inaccessible (refer to section 8.1.1.6);
• ANA Persistent Loss (refer to section 8.1.1.7); and
• ANA Change (refer to section 8.1.1.8).
 ANA Optimized state
While the relationship between the controller and an ANA group is in this state, the characteristic of that
relationship to each namespace in that group is optimized. Commands processed by a controller that
reports this state for an ANA Group provide optimized access characteristics to any namespace in that ANA
Group. A controller that supports ANA Reporting shall support reporting this state.
While in this state, all commands, functions, and operations supported by the namespace shall perform as
described in this specification.
 ANA Non-Optimized state
While the relationship between the controller and an ANA group is in this state, the characteristic of that
relationship to each namespace in that group is non -optimized. Commands processed by a controller that
reports this state for an ANA Group provide n on-optimized access characteristics (e.g., the processing of
some commands, especially those involving data transfer, may operate with lower performance or may use
NVM subsystem resources less effectively than if a controller is used that reports the optim ized state) to
any namespace in that ANA Group. Support for reporting this state is optional.
While in this state, all commands, functions, and operations supported by the namespace shall perform as
described in this specification.
