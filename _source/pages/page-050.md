---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 50
headings: []
---

# Source page 50

NVM Express® Base Specification, Revision 2.1

28
Figure 14: Complex NVM Storage Hierarchy with NVM Sets
NVM Subsystem
Domain A
Endurance Group A1
NVM Set A1b
NS NS
NVM Set A1e
NS NS
Endurance Group A3
NVM Set A3f
NS NS
NVM Set A3j
NS NS
Domain D
Endurance Group D1
NVM Set D1k
NS NS
NVM Set D1m
NS NS
Endurance Group D5
NVM Set D5n
NS NS
NVM Set D5p
NS NS
... ... ... ...
... ... ... ...
...
...
...
...
... ...
...

Entity naming key (Abc):
A: Domain (capital letter)
b: Endurance Group (digit)
c: NVM Set (lower case letter)
Figure 15 shows the relationships in a complex NVM subsystem, which has:
• multiple domains;
• multiple Endurance Groups per domain;
• multiple Reclaim Groups per Endurance Group; and
• multiple namespaces per Endurance Group.
The placement (i.e., which Reclaim Group) of user data for a namespace is directed by each host write
command to that namespace (refer to section 8.1.10).
