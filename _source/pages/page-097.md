---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 97
headings: ["3.2 NVM Subsystem Entities", "3.2.1 Namespaces", "3.2.1.1 Namespace Overview", "3.2.1.2 Valid and Invalid NSIDs", "3.2.1.3 Allocated and Unallocated NSID Types", "3.2.1.4 Active and Inactive NSID Types", "3.2.1.5 NSID and Namespace Relationships"]
---

# Source page 97

NVM Express® Base Specification, Revision 2.1

75
3.2 NVM Subsystem Entities
 Namespaces
 Namespace Overview
A namespace is a formatted quantity of non -volatile memory that may be directly accessed by a host . A
namespace ID (NSID) is an identifier used by a controller to provide access to a namespace.
 Valid and Invalid NSIDs
Valid NSIDs are the range of possible NSIDs that may be used to refer to namespaces that exist in the
NVM subsystem. Any NSID is valid, except if that NSID is 0h or greater than the Number of Namespaces
field reported in the Identify Controller data structure (refer to Figure 312). NSID FFFFFFFFh is a broadcast
value that is used to specify all namespaces. An invalid NSID is any value that is not a valid NSID and is
also not the broadcast value.
Valid NSIDs are:
a) allocated or unallocated in the NVM subsystem; and
b) active or inactive for a specific controller.
 Allocated and Unallocated NSID Types
In the NVM subsystem, a valid NSID is:
a) an allocated NSID; or
b) an unallocated NSID.
Allocated NSIDs refer to namespaces that exist in the NVM subsystem. Unallocated NSIDs do not refer to
any namespaces that exist in the NVM subsystem.
 Active and Inactive NSID Types
For a specific controller, an allocated NSID is:
a) an active NSID; or
b) an inactive NSID.
Active NSIDs for a controller refer to namespaces that are attached to that controller. Allocated NSIDs that
are inactive for a controller refer to namespaces that are not attached to that controller.
Unallocated NSIDs are inactive NSIDs for all controllers in the NVM subsystem.
An allocated NSID may be an active NSID for some controllers and an inactive NSID for other controllers
in the same NVM subsystem if the namespace that the NSID refers to is attached to some controllers, but
not all controllers, in the NVM subsystem.
Refer to section 8.1.15 for actions associated with a namespace being detached or deleted.
 NSID and Namespace Relationships
Unless otherwise noted, specifying an inactive NSID in a command that uses the Namespace Identifier
(NSID) field shall cause the controller to abort the command with a status code of Invalid Field in Command.
Specifying an invalid NSID in a command that uses the NSID field shall cause the controller to abort the
command with a status code of Invalid Namespace or Format.
Figure 65 summarizes the valid NSID types and Figure 66 visually shows the NSID types and how they
relate.
Figure 65: NSID Types and Relationship to Namespace
Valid NSID Type NSID relationship to namespace Reference
Unallocated Does not refer to any namespace that exists in the NVM subsystem 3.2.1.3
