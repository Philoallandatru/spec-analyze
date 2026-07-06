---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 495
headings: []
---

# Source page 495

NVM Express® Base Specification, Revision 2.1

473
Figure 581: Reservation Status Data Structure
Bytes Description
09
Persist Through Power Loss State (PTPLS): This field indicates the Persist Through Power Loss
State associated with the namespace.
Value Definition
0 Reservations are released and registrants are cleared on a power on.
1 Reservations and registrants persist across a power loss.

23:10 Reserved
Registered Controller Data Structure List
47:24
Registrant Data Structure 0
Note: This field was formerly named Registered Controller Data Structure 0.
… …
24*n+47:
24*(n+1) Registrant Data Structure n

Figure 582: Reservation Status Extended Data Structure
Bytes Description
Header
23:00 Reservation Status Header (RSHDR): Refer to the Reservation Status Header in Figure 581 for
definition.
63:24 Reserved
Registered Controller Extended Data Structure List
127:64
Registrant Extended Data Structure 0
Note: This field was formerly named Registered Controller Extended Data Structure 0.
… …
64*(n+1)+63:
64*(n+1) Registrant Extended Data Structure n

Figure 583: Registered Controller Data Structure
Bytes Description
01:00
Controller ID (CNTLID): If a registrant of the namespace is associated with a controller in the NVM
subsystem, then this field contains the controller ID (i.e., the value of the CNTLID field in the Identify
Controller data structure) of the controller that is associated with the registrant whose host identifier
is indicated in the Host identifier field of this Registrant data structure.
If a registrant of the namespace is not associated with any controller in the NVM subsystem, then the
controller processing the command shall set this field to FFFFh.
If the namespace is a dispersed namespace and the controller is not contained in the same
participating NVM subsystem as the controller processing the command, then the Controller ID field
is set to FFFDh, as described in section 8.1.9.6.
02
Reservation Status (RCSTS): This field indicates the reservation status of the registrant associated
with this data structure (i.e., the registrant whose host identifier is indicated in the Host Identifier field
of this Registrant data structure).
Bits Description
7:1 Reserved
0
Association with Host Holding Reservation (AHHR):  If this bit is set to '1', then the
registrant associated with this data structure holds a reservation on the namespace. If
this bit is cleared to ‘0’, then the controller is not associated with a host that holds a
reservation on the namespace.

07:03 Reserved
15:08 Host Identifier (HOSTID):  This field contains the 64-bit Host Identifier of the registrant of the
namespace described by this data structure.
