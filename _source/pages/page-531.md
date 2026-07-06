---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 531
headings: ["8.1.8.3.2.2 Release Resources (Directive Operation 02h)", "8.1.8.4 Data Placement (Directive Type 02h, Optional)", "8.1.9 Dispersed Namespaces"]
---

# Source page 531

NVM Express® Base Specification, Revision 2.1

509
future operation, then that stream identifier is referring to a different stream. If the specified identifier does
not correspond to an open stream for the specified namespace, then the Directive Send command should
not fail as a result of the specified identifier. If there are stream resources allocated for the exclusive use of
the specified namespace, then those exclusive stream resources remain allocated for this namespace and
may be re-used in a subsequent write command. If there are no stream resources allocated for the exclusive
use of the specified namespace, then the stream resources ar e returned to the NVM subsystem stream
resources for future use by a namespace without exclusive allocated stream resources. If an NSID value
of FFFFFFFFh is specified, then the controller shall abort the command with a status  code of Invalid Field
in Command.
No data transfer occurs.
 Release Resources (Directive Operation 02h)
The Release Resources operation is used to release all streams resources allocated for the exclusive use
of the namespace attached to all controllers:
• associated with the same non -zero Host Identifier of the controller that processed the operation if
the SSID bit is cleared to ‘0’ in the NSSC field is cleared to ‘0’; and
• associated with any non-zero Host Identifier if the SSID bit is set to ‘1’.
On successful completion of this command, the exclusive allocated stream resources are released and the
Namespace Streams Allocated (refer to Figure 607) field is cleared to 0h for the specified namespace. If
this command is issued when no streams resources are allocated for the exclusive use of the namespace,
then the Directive Send command shall take no action and shall not fail as a result of no allocated stream
resources.
No data transfer occurs.
 Data Placement (Directive Type 02h, Optional)
The Data Placement Directive enables the host to specify to the controller the Reclaim Unit (refer section
8.1.10) to place the user data in I/O commands specified by the appropriate I/O Command Set specification.
The Data Placement Directive has no Directive Operations defined. Any Directive Receive command or
Directive Send command that specifies a Data Placement Directive Type shall be aborted by the controller
with a status code of Invalid Field in Command.
If the specified namespace is not contained in an Endurance Group with Flexible Data Placement enabled,
then the controller shall abort the Directive Send command with a status code of FDP Disabled.
 Dispersed Namespaces
Dispersed namespaces provide hosts with access to the same namespace using multiple participating NVM
subsystems. A typical Dispersed Namespace implementation uses Fabrics based NVM subsystems. Two
prominent scenarios that require namespace access from multiple participating NVM subsystems are:
1. online data migration; and
2. data replication.
Online data migration is used to transparently move the contents of one or more namespaces across
participating NVM subsystems without disrupting host access to those namespaces during the migration.
For each namespace whose contents are migrated, one or more paths to the namespace in the destination
NVM subsystem are added to the multi -path I/O on the host, in addition to the existing paths to the
namespace in the source NVM subsystem. Once the online data migration completes (i.e., all of the
contents of all namespaces being migrated has been successfully migrated from the source NVM
subsystem to the destination NVM subsystem), all of the paths to the namespace in the source NVM
subsystem are subsequently removed from the multi-path I/O on the host. During the online data migration,
the host and participating NVM subsystems must manage the paths so that there is no interruption to I/O.
Figure 611 shows an example of online data migration, where the contents of namespace B are migrated
from one participating NVM subsystem to another participating NVM subsystem. In this figure, the lines
