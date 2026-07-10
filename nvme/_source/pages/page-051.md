---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 51
headings: []
---

# Source page 51

NVM Express® Base Specification, Revision 2.1

29
Figure 15: Complex NVM Storage Hierarchy with Multiple Reclaim Groups

Entity naming key (Abc):
A: Domain (capital letter)
b: Endurance Group (digit)
c: Reclaim Group (digit or lower-case letter for maximum number)
The support of Endurance Groups , Reclaim Groups within an Endurance Group, or NVM Sets  within an
Endurance Group is optional, but the storage model supports these concepts. An NVM subsystem may be
shipped by the vendor with storage entities configured, or an NVM subsystem may be configured or re -
configured by the customer. Typical changes to the configuration are creation and deletion of namespaces.
An NVM subsystem that does not support multiple NVM Sets does not require reporting of NVM Sets. An
NVM subsystem that does not support multiple Endurance Groups does not require reporting of Endurance
Groups.
NVM Subsystem
...
Domain A
Endurance Group A1
Reclaim
Group
A10
Reclaim
Group
A11
Reclaim
Group
A1f
Namespace
...
Namespace
Endurance Group A3
Reclaim
Group
A30
Reclaim
Group
A31
Reclaim
Group
A3g
Namespace
...
Namespace
...
Domain D
Endurance Group D1
Reclaim
Group
D10
Reclaim
Group
D11
Reclaim
Group
D1h
Namespace
...
Namespace
Endurance Group D5
Reclaim
Group
D50
Reclaim
Group
D51
Reclaim
Group
D5i
Namespace
...
Namespace...
