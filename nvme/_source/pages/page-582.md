---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 582
headings: ["8.1.22.1 Reservation Types"]
---

# Source page 582

NVM Express® Base Specification, Revision 2.1

560
Support for reservations by a namespace or controller is optional. A namespace indicates support for
reservations by reporting a non -zero value in the Reservation Capabilities (RESCAP) field in the Identify
Namespace data structure. A controller indicates support for reservations through the Optional NVM
Command Support (ONCS) field in the Identify Controller data structure  (refer to Figure 312). If a host
submits a command associated with reservations (i.e., Reservation Report, Reservation Register,
Reservation Acquire, and Reservation Release) to a controller or a namespace that do not both support
reservations, then the command is aborted by the controller with a status code of Invalid Command Opcode.
Controllers that make up an NVM subsystem shall all have the same support for reservations. Although
strongly encouraged, namespaces that make up an NVM subsystem are not all required to have the same
support for reservations. For example, some namespaces within a single controller may support
reservations while others do not, or the supported reservation types may differ among namespaces. If a
controller supports reservations, then the controller shall:
• Indicate support for reservations by returning a '1' in the Reservations Support (RESERVS) bit of
the Optional NVM Command Support (ONCS) field in the Identify Controller data structure;
• Support the Reservation Report command (refer to section 7.8), Reservation Register command
(refer to section 7.6), Reservation Acquire command (refer to section 7.5), and Reservation
Release command (refer to section 7.7);
• Support the Reservation Notification log page;
• Support the Reservation Log Page Available asynchronous events;
• Support the Reservation Notification Mask Feature;
• Support the Host Identifier Feature; and
• Support the Reservation Persistence Feature.
If a controller supports dispersed namespaces and supports reservations, then the controller supports the
Dispersed Namespace Reservation Support (DISNSRS) bit being set to ‘1’ in the Reservation Report
command, Reservation Register command, Reservation Ac quire command, and Reservation Release
command, as described in section 8.1.9.6.
If a namespace supports reservations, then the namespace shall:
• Report a non-zero value in the Reservation Capabilities (RESCAP) field in the Identify Namespace
data structure;
• Support Persist Through Power Loss (PTPL) state; and
• Support sufficient resources to allow a host to successfully register a reservation key on every
controller in the NVM subsystem with access to the shared namespace (i.e., a Reservation Register
command shall never fail due to lack of resources).
NOTE: The behavior of Ignore Existing Key has been changed to improve compatibility with SCSI based
implementations. Conformance to the modified behavior is indicated in the Reservation Capabilities
(RESCAP) field of the Identify Namespace data structure. For the previous definition of Ignore Existing Key
behavior, refer to NVM Express Base Specification, Revision 1.2.1.
 Reservation Types
The NVM Express interface supports six types of reservations:
• Write Exclusive;
• Exclusive Access;
• Write Exclusive - Registrants Only;
• Exclusive Access - Registrants Only;
• Write Exclusive - All Registrants; and
• Exclusive Access - All Registrants.
