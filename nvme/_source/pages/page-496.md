---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 496
headings: ["7.8.1 Command Completion"]
---

# Source page 496

NVM Express® Base Specification, Revision 2.1

474
Figure 583: Registered Controller Data Structure
Bytes Description
23:16 Reservation Key (RKEY): This field contains the reservation key of the registrant described by this
data structure.

Figure 584: Registered Controller Extended Data Structure
Bytes Description
01:00 Controller ID (CNTLID): Refer to Figure 583 for definition.
02 Reservation Status (RCSTS): Refer to Figure 583 for definition.
07:03 Reserved
15:08 Reservation Key (RKEY): Refer to Figure 583 for definition.
31:16 Host Identifier (HOSTID):  This field contains the 128-bit Host Identifier of the registrant of the
namespace described by this data structure.
63:32 Reserved
 Command Completion
When the command is completed, the controller shall post a completion queue entry to the associated I/O
Completion Queue indicating the status for the command.
