---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 526
headings: []
---

# Source page 526

NVM Express® Base Specification, Revision 2.1

504
that has registered a different Host Identifier is referencing the same stream). All controllers in an NVM
subsystem shall report the same value in the NSSC field.
If multiple controllers receive a registration of a Host Identifier (refer to section 5.1.25.1.28) that has the
same non-zero value, then that value represents a single host that is accessing the namespace through
those controllers and a stream identifier is used across those controllers to access the same stream on the
namespace. If a Host Identifier has a unique non-zero value, then each value represents a unique host that
is accessing the namespace and:
• if the SSID bit is cleared to ‘0’, then the same stream identifier on controllers with different non-zero
Host Identifiers does not have the same meaning for a particular namespace (i.e., the stream
identifier is not used across controllers with different non -zero Host Identifiers to access the same
stream on the namespace); and
• if the SSID bit is set to ‘1’, then the same stream identifier on any controller with a non -zero Host
Identifier has the same meaning for a particular namespace (i.e., the stream identifier is used
across controllers to access the same stream on the namespace).
If a Host Identifier is cleared to 0h, then a unique host is accessing the namespace and the stream identifier
does not have the same meaning for a particular namespace.
Figure 605: Example Multi-Stream and NSSC
Namespace
NSID 1
NVM Express
Controller 1
Host ID = A
NSID 1
NVM Express
Controller 2
Host ID = A
NSID 1
NVM Express
Controller 3
Host ID = B
NSID 1
Host
A
Host
B
Host
C
NVM Subsystem
NVM Express
Controller 4
Host ID = C
Stream
ID 1-a
Stream
ID 1-b
Stream
ID 1-c Stream
ID 1-d

In the example shown in Figure 605, if the SSID bit is cleared to ‘0’, then there are three streams as follows:
• Stream ID 1-a and Stream ID 1-b have the same meaning;
• Stream ID 1-c has a different meaning; and
• Stream ID 1-d has a different meaning.
In the example shown in Figure 605, if the SSID bit is set to ‘1’, then there is one stream as follows:
• Stream ID 1-a, Stream ID 1-b, Stream ID 1-c, and Stream ID 1-d have the same meaning.
The controller(s) recognized by the NVM subsystem as being associated with a specific host or hosts and
attached to a specific namespace either:
• utilizes a number of stream resources allocated for exclusive use of that namespace as returned
in response to an Allocate Resources operation; or
• utilizes resources from the NVM subsystem stream resources.
