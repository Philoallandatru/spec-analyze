---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 169
headings: ["4.2.3.3 Media and Data Integrity Errors Definition"]
---

# Source page 169

NVM Express® Base Specification, Revision 2.1

147
Figure 104: Status Code – Command Specific Status Values, I/O Commands
Value Description
98h Program Not Activated
99h Program In Use
9Ah Program Index Not Downloadable
9Bh Program Too Big
9Ch Successful Media Verification Read
9Dh to B7h Reserved
B8h Zoned Boundary Error
B9h Zone Is Full
BAh Zone Is Read Only
BBh Zone Is Offline
BCh Zone Invalid Write
BDh Too Many Active Zones
BEh Too Many Open Zones
BFh Invalid Zone State Transition

Figure 105: Status Code – Command Specific Status Values, Fabrics Commands
Value Description Commands
Affected
80h Incompatible Format:  The NVM subsystem does not support the record format
specified by the host.
Connect,
Disconnect
81h
Controller Busy: The controller is already associated with a host (Connect
command). This value is also returned if there is no available controller (Connect
command).
The controller is not able to disconnect the I/O Queue at the current time (Disconnect
command).
Connect,
Disconnect
82h
Connect Invalid Parameters: One or more of the command parameters (e.g., Host
NQN, NVM Subsystem NQN, Host Identifier, Controller ID, Queue ID) specified are
not valid.
Connect
83h Connect Restart Discovery: The NVM subsystem requested is not available. The
host should restart the discovery process. Connect
84h
Connect Invalid Host: The host is not allowed to establish an association to any
controller in the NVM subsystem or the host is not allowed to establish an
association to the specified controller.
Connect
85h Invalid Queue Type: The command was sent on the wrong queue type (e.g., a
Disconnect command was sent on the Admin queue). Disconnect
86h to 8Fh Reserved
90h
Discover Restart: The snapshot of the records is now invalid or out of date. If the
Discovery log page was requested, then the host or Discovery controller should re-
read the Discovery log page. If the Host Discovery log page was requested, then
the host or Discovery controller should re-read the Host Discovery log page.
Get Log Page
91h Authentication Required: NVMe in-band authentication is required and the queue
has not yet been authenticated. NOTE 1
92h to AFh Reserved
B0h to BFh Transport Specific:  The status values in this range are NVMe Transport specific. Refer to the
appropriate NVMe Transport binding specification for the definition of these status values.
Notes:
1. All commands other than Connect, Authenticate Send, and Authenticate Receive.
 Media and Data Integrity Errors Definition
Completion queue entries with a Status Code Type (SCT) of Media and Data Integrity Errors indicate an
error associated with the command that is due to an error associated with the NVM media or a data integrity
type error.
