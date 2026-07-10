---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 581
headings: []
---

# Source page 581

NVM Express® Base Specification, Revision 2.1

559
rights for that command with respect to existing reservations, if any), then that command shall not be
subsequently aborted by the controller with a status code of Reservation Conflict (e.g., due to a subsequent
reservation).
Figure 644: Example Multi-Host System
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

A reservation requires an association between a host and a namespace. As shown in Figure 644, each
controller in a multi-path I/O and namespace sharing environment is associated with exactly one host. While
it is possible to construct systems where two or more hosts share a single controller, such usage is outside
the scope of this specification.
A host may be associated with multiple controllers. In Figure 644 host A is associated with two controllers
while hosts B and C are each associated with a single controller. A host should register a non-zero Host
Identifier (refer to section 5.1.25.1.28) with each controller with which that host is associated using a Set
Features command (refer to section 5.1.25) prior to performing any operations associated with reservations.
The Host Identifier allows the NVM subsystem to identify controllers associated with the same host and
preserve reservation properties across these controllers (i.e., a host issued command has the same
reservation rights no matter which controller associated with the host processes the command).
An NVM subsystem may require that the host register a non -zero Host Identifier for a host to use
reservations (i.e., the RHII bit is set to ‘1’ in the CTRATT field of the Identify Controller data structure (refer
to Figure 312)). If the controller does not support reservations with a Host Identifier value of 0h and a
reservation command is received from a host with a Host Identifier value of 0h, then the controller shall
abort that reservation command with a status of Host Identifier Not Initialized.
If an NVM subsystem:
1. supports reservations with a Host Identifier value of 0h;
2. registrations or reservations are established by a host with a Host Identifier value of 0h; and
3. the Host Identifier is changed to a non-zero value,
then those registrations or reservations remain associated with the Host with a Host Identifier value of 0h
and are not associated with the host with the non-zero Host Identifier.
If the controller does not support reservations with a Host Identifier value of 0h, reservations may have
been established by hosts with non -zero Host Identifiers connected to other controllers, and commands
from a host with a Host Identifier value of 0h that conflict with a reservation (refer to Figure 645 and Figure
646) are aborted by the controller with a status code of Reservation Conflict.
