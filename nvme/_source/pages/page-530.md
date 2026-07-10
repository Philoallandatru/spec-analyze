---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 530
headings: ["8.1.8.3.1.3 Allocate Resources (Directive Operation 03h)", "8.1.8.3.2 Directive Send", "8.1.8.3.2.1 Release Identifier (Directive Operation 01h)"]
---

# Source page 530

NVM Express® Base Specification, Revision 2.1

508
Figure 608: Streams Directive – Get Status Data Structure
Bytes Description
01:00 Open Stream Count (OSC): This field specifies the number of streams that are currently open.
03:02 Stream Identifier 1 (SID1): This field specifies the stream identifier of the first (numerically lowest)
open stream.
05:04 Stream Identifier 2 (SID2): This field specifies the stream identifier of the second open stream.
… …
131071:
131070
Stream Identifier 65,535 (SID655355): This field specifies the stream identifier of the 65,535th
open stream.
 Allocate Resources (Directive Operation 03h)
The Allocate Resources operation indicates the number of streams that the host requests for the exclusive
use for the specified namespace. If the SSID bit is cleared to ‘0’ in the NSSC field, then those resources
are for the exclusive use of hosts that are registered with the same Host Identifier as the host that made
the request. If the SSID bit is set to ‘1’, then those resources are for the exclusive use of any host that is
registered with any non -zero Host Identifier. The DSPEC field in command Dword 11  is not used for this
operation. The operation returns the number of streams allocated in Dword 0 of the completion queue entry.
The value allocated may be less than or equal to the number requested. The allocated resources shall be
reflected in the Namespace Streams Allocated field of the Return Parameters data structure.
If the controller is unable to allocate any stream resources for the exclusive use for the specified
namespace, then the controller shall:
• return a status value of Stream Resource Allocation Failed; or
• if NVM subsystem stream resources are available, then clear NSA to 0h in the completion queue
entry to indicate that the host may use stream resources from the NVM subsystem for this
namespace.
If the specified namespace already has stream resources allocated for the exclusive use of the host issuing
the Allocate Resources operation, then the controller shall return a status code of Invalid Field in Command.
To allocate additional streams resources, the host should release resources and request a complete set of
resources.
No data transfer occurs.
Figure 609: Allocate Resources – Command Dword 12
Bits Description
31:16 Reserved
15:00 Namespace Streams Requested (NSR): This field specifies the number of stream resources the host
is requesting be allocated for exclusive use by the specified namespace.

Figure 610: Allocate Resources – Completion Queue Entry Dword 0
Bits Description
31:16 Reserved
15:00
Namespace Streams Allocated (NSA): This field indicates the number of streams resources that have
been allocated for exclusive use by the specified  namespace. The allocated resources are available to
all controllers associated with that host.
8.1.8.3.2 Directive Send
This section defines operations used with the Directive Send command for the Streams Directive.
 Release Identifier (Directive Operation 01h)
The Release Identifier operation specifies that the stream identifier specified in the DSPEC field in
command Dword 11 is no longer in use by the host. Specifically, if the host uses that stream identifier in a
