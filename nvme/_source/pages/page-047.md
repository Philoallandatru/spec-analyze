---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 47
headings: []
---

# Source page 47

NVM Express® Base Specification, Revision 2.1

25
Each Endurance Group is composed of storage media, which are termed Media Units (refer to section
8.1.4.2) or Reclaim Units (refer to section 3.2.4). Reclaim Unit Handles reference a Reclaim Unit in each
Reclaim group for writing user data. For clarity, Media Units, Reclaim Unit Handles, and Reclaim Units are
not shown in the examples in this section that follow.
Figure 11 shows the hierarchical relationships of these entities within a simple NVM subsystem, which has:
• one domain;
• one Endurance Group;
• one NVM Set; and
• one namespace.
Figure 11: Simple NVM Storage Hierarchy with NVM Sets
NVM Subsystem
Domain 0
Endurance Group
NVM Set
Namespace 1
