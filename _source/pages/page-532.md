---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 532
headings: []
---

# Source page 532

NVM Express® Base Specification, Revision 2.1

510
from the host to the NVMe controllers may represent multiple connections from the host to multiple
controllers in each participating NVM subsystem.
Figure 611: Online Data Migration

Data replication is used to replicate the contents of one or more dispersed namespaces across participating
NVM subsystems such that each participating NVM subsystem is able to provide host access to the same
namespace data. Other behavior (e.g., persisten t reservations) must also be maintained between hosts
and each participating NVM subsystem. Figure 612, Figure 613, and Figure 614 show different examples
of data replication, where the contents of namespace B are replicated across two participating NVM
subsystems. In these figures, the lines from hosts to NVMe controllers may represent multiple connections
from the hosts to multiple controllers in each participating NVM subsystem. Figure 612 shows an example
of single NVM subsystem host connectivity, where two hosts access the same dispersed namespace being
replicated across two participating NVM subsystems via controllers in only one NVM subsystem.
NSID 1

NSID 2

NVMe Controller(s)
NS A

NSID 1

NSID 3

NVMe Controller(s)
NS C
Host
NS B
NVM Subsystem 1
 NVM Subsystem 2
