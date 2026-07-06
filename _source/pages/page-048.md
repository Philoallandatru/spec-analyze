---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 48
headings: []
---

# Source page 48

NVM Express® Base Specification, Revision 2.1

26
Figure 12 shows the hierarchical relationships in a simple NVM subsystem, which has:
• one domain;
• one Endurance Group;
• one Reclaim Group; and
• one namespace with user data written to the single Reclaim Group.
The placement (i.e., which Reclaim Group) of user data for a namespace is directed by each host write
command to that namespace (refer to section 8.1.10).
Figure 12: Simple NVM Storage Hierarchy with One Reclaim Group

Figure 13 shows the hierarchical relationships in a simple NVM subsystem, which has:
• one domain;
• one Endurance Group;
• four Reclaim Groups; and
• one namespace with user data written to each Reclaim Group.
The placement (i.e., which Reclaim Group) of user data for a namespace is directed by each host write
command to that namespace (refer to section 8.1.10).
NVM Subsystem
Domain 0
Endurance Group 1
Reclaim
Group 0
Namespace 1
