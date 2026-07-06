---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 659
headings: []
---

# Source page 659

NVM Express® Base Specification, Revision 2.1

637
• Associate individual Underlying Namespaces with an Exported NVM Subsystem by issuing a
Manage Exported Namespace command with an Associate Namespace operation (refer to section
5.3.7.1.1) (refer to step ‘4’ in Figure 719 and Figure 720).
• Associate individual Underlying Ports with an Exported NVM Subsystem by issuing a Manage
Exported Port command with a Create operation (refer to section 5.3.9.1.1) (refer to step ‘5’ in
Figure 719 and Figure 720).
Figure 719: Example reference for retrieving Underlying Namespaces and Underlying Ports
Underlying Namespace
List
NVM Storage
NVMe
SSD
1
Underlying NamespaceUnderlying Namespace
Underlying Port
(Ethernet)
Underlying Port
(Fibre channel)
Underlying Port
(Infiniband)
2
Underlying Ports List
NVMe
SSDNVMe
SSD
Exported NVM resource collection
Exported NVM Subsystem (SUBNQN)
Exported Namespace
Exported Port ID
Allowed
Host List
3
6
4
5
