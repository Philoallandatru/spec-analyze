---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 660
headings: ["8.3.3.1.2 Managing Host access to an Exported NVM Subsystem"]
---

# Source page 660

NVM Express® Base Specification, Revision 2.1

638
Figure 720: Example steps for retrieving underlying resources and configuring an Exported
NVM Subsystem
Administrator
Controller
1
2
3
4
5

8.3.3.1.2 Managing Host access to an Exported NVM Subsystem
Exported NVM Subsystems are created with a specified Access Mode (i.e., restricted, unrestricted).
This section describes an example flow for changing an Exported NVM Subsystem configured for
unrestricted access to restricted access (where only hosts in the Allowed Host List associated to the
specified Exported NVM Subsystem have permissions to use the Exported Namespaces linked to that
Exported NVM Subsystem).
The access mode of Exported NVM Subsystems may be changed by issuing a Manage Exported NVM
Subsystem command with a Change Access Mode operation (refer to section 5.3.8.1.2) and specifying the
desired Access Mode (refer to step ‘6’ in Figure 721). The example Figure 721 shows changing the access
mode of an Exported NVM Subsystem to restricted access. The example Figure 721 also shows Exported
NVM Subsystem access to specific hosts (refer to step ‘7’ in Figure 721).
Figure 721: Example steps for restricting access to an Exported NVM Subsystem to specific hosts
Administrator
Controller
6
7

Similarly, hosts may be restricted from access to specified Exported NVM Subsystems configured for
restricted access, on specified ports, by issuing Manage Exported NVM Subsystem commands with the
Revoke Host Access operation (refer to section 5.3.8.1.4). Refer to step ‘8’ in Figure 722).
