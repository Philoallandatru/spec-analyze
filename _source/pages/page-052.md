---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 52
headings: ["2.3.2 I/O Command Sets", "2.3.3 NVM Subsystem Examples"]
---

# Source page 52

NVM Express® Base Specification, Revision 2.1

30
 I/O Command Sets
I/O commands perform operations on namespaces , and each namespace is associated with exactly one
I/O command set . For example, commands in the NVM Command Set access data represented in a
namespace as logical blocks, and commands in the Key  Value Command Set access data represented in
a namespace as key-value pairs.
The association of a namespace to an  I/O command set is specified when the namespace is created  and
is fixed for the lifetime of that namespace.
A controller may support one or more I/O command sets. Namespaces that are associated with  the I/O
command sets that are supported and enabled on a controller may be attached to that controller. A host
issues commands to a namespace and those commands are interpreted based on the I/O command set
associated with that namespace.
 NVM Subsystem Examples
Figure 16 illustrates a simple NVM subsystem that has a single instance of each storage entity.
Figure 16: Single-Namespace NVM Subsystem

NVM Subsystem (SSD)
Domain 0
Endurance Group 1
Port
NVM Set 1
Controller
Namespace 1
• The NVM subsystem consists of a single port and a single domain.
• The domain contains a controller and storage media.
• All of the storage media are contained in one Endurance Group.
• All of the storage media in that Endurance Group are organized into one NVM Set.
• That NVM Set contains a single namespace.
