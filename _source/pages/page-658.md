---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 658
headings: ["8.3.3.1.1 Configuring an Exported NVM Subsystem"]
---

# Source page 658

NVM Express® Base Specification, Revision 2.1

636
• Manage Exported Namespace command to configure an Exported NVM Subsystem with
namespaces;
• Manage Exported Port command to manage fabric port configurations on an Exported NVM
Subsystem;
• Manage Exported NVM Subsystem command to delete Exported NVM Subsystems; and
• Clear Exported NVM Resource Configuration command to remove all configured Exported NVM
Resources.
8.3.3.1.1 Configuring an Exported NVM Subsystem
Prior to configuring exported NVM resources and exposing them for access, an administrative entity (e.g.,
administrator, resource manager, orchestrator, administration console, centralized configuration manager)
must first determine what Underlying resourc es (e.g., namespaces, ports) are available; this may be
through a-priori knowledge or by using a Identify command with CNS set to 1Dh (refer to section 5.1.13.4.1
(refer to step ‘1’ in Figure 718, Figure 719, and in Figure 720) and Identify command with CNS set to 1Eh
(refer to section 5.1.13.4.2) (refer to step ‘2’ in Figure 718, Figure 719, and Figure 720). Figure 718 shows
collections of Underlying Namespaces and Underlying Ports and an empty collection of Exported NVM
resources.
Figure 718: Example reference for retrieving Underlying Namespaces and Underlying Ports
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

Once the administrative entity has determined the available underlying resources (refer to step ‘1’ and step
‘2’ in Figure 718, Figure 719, and Figure 720), the administrative entity may create, configure, and expose
Exported NVM Subsystems. An example flow to configure an Exported NVM Subsystem:
• Create an Exported NVM Subsystem by Issuing a Create Exported NVM Subsystem command
(refer to section 5.3.2) (refer to step ‘3’ in Figure 719 and Figure 720).
