---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 534
headings: ["8.1.9.1 Dispersed Namespace Management", "8.1.9.2 NSID and Globally Unique Namespace Identifier Usage"]
---

# Source page 534

NVM Express® Base Specification, Revision 2.1

512
Figure 614: Data Replication Example 3

 Dispersed Namespace Management
The Namespace Management command (refer to section 5.1.21) may be used to create a namespace that
is capable of being accessed using controllers contained in two or more participating NVM subsystems
concurrently (i.e., may be used to create a namespace that is capable of being a dispersed namespace).
The methods  for dispersing a namespace (e.g., attaching that namespace to controllers contained in
separate participating NVM subsystems) are outside the scope of this specification. The Namespace
Management or Capacity Management command may be used to delete a disp ersed namespace on the
participating NVM subsystem containing the controller processing the command. Deleting a dispersed
namespace on one participating NVM subsystem may or may not affect that namespace on other
participating NVM subsystems (e.g., a deletion from one participating NVM subsystem may have no effect
on the namespace on other participating NVM subsystems, or a deletion from one participating NVM
subsystem may delete the namespace from all participating NVM subsystems).
The Namespace Management command may be used to create a shared namespace that is not a
dispersed namespace which is later converted to a dispersed namespace. The method of converting a
namespace that is not a dispersed namespace to a dispersed namespace i s outside the scope of this
specification. NVM subsystems should not convert private namespaces to dispersed namespaces.
Whenever a namespace that is not a dispersed namespace is converted to a dispersed namespace or a
dispersed namespace is converted to a  namespace that is not a dispersed namespace, the controller
reports a Namespace Attribute Changed event as described in Figure 151.
The host may attach or detach a dispersed namespace from the controller by using the Namespace
Attachment command (refer to section 5.1.20).
 NSID and Globally Unique Namespace Identifier Usage
The NSID for a dispersed namespace is unique for all controllers in a participating NVM subsystem, as
described in section 3.2.1.6. The NSID for a dispersed namespace may be different on different
participating NVM subsystems.
A dispersed namespace has a globally unique namespace identifier (refer to section 4.7.1.4) that uniquely
identifies the namespace in all participating NVM subsystems. The globally unique namespace identifier
has the same value for that namespace in all participating NVM subsystems and shall be used to determine
which NSIDs on different participating NVM subsystems refer to the same namespace. The globally unique
NSID 1

NSID 2

NVMe Controller(s)
NS A

NSID 1

NSID 3

NVMe Controller(s)
NS C
NS B
NVM Subsystem 1
 NVM Subsystem 2
Host 1
 Host 2
