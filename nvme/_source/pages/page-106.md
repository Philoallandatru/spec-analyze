---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 106
headings: ["3.2.5.2 Domains and Reservations", "3.2.5.3 Domain Identifier Use (Informative)"]
---

# Source page 106

NVM Express® Base Specification, Revision 2.1

84
controllers and no NVM storage capacity. Domain 4 is a domain that contains no controllers and contains
some amount of NVM storage capacity from which NVM Endurance Groups have been created that contain
two shared namespaces (i.e., NS D and NS E ). Domain 5 is a domain that contains one controller and no
NVM storage capacity. Domain 6 is a domain that contains no controllers and no NVM storage capacity
allocated to an NVM Endurance Group (i.e., an empty domain).
Figure 72: Example 2 Domain Structure
NSID 1 NSID 2
Fabric
NVMe Controller
NS
A
NS
B
Port v Port w
Fabric
NVMe Controller
Fabric
NVMe Controller
Fabric
NVMe Controller
Fabric
NVMe Controller
Port x Port y Port z
Domain 2 Domain 4
Communication
boundary
Communication
boundary
NSID 2 NSID 2NSID 2 NSID 2NSID 4 NSID 4NSID 4
NS
D
Communication
boundary
Communication
boundary
Domain 1 Domain 3 Domain 5
NS
C

NSID 3
Domain 6
NS
E
Key: - - - - (Dashed Line) – Communication Boundary

 Domains and Reservations
If an NVM subsystem supports multiple domains and Persistent Reservations (refer to section 8.1.22), then
resumption after a division event (e.g., resumption of operation, resumption of communication) requires
that all persistent reservation state within the domains in the NVM subsystem that are no longer divided be
synchronized (i.e., updated).
If the reservation state for a namespace is not synchronized, then the ANA Group that contains that
namespace shall transition to the ANA Inaccessible state (refer to section 8.1.1.6) and remain in that state
until the Persistent Reservation state is synchronized. If the Persistent Reservation state is not able to be
synchronized, then:
• a transition to the ANA Persistent Loss state occurs and commands are processed as described in
section 8.1.1.7; or
• the controller may stop processing commands and set the Controller Fatal Status (CSTS.CFS) bit
to ‘1’ (refer to section 9.5).
 Domain Identifier Use (Informative)
Domain Identifier values indicate the parts of the NVM subsystem that comprise a domain.
The host may use these values to determine which Endurance Groups (refer to section 3.2.3) are contained
in the same domain and which are contained in a different domain. Examples of host use of the domain
identifier include:
