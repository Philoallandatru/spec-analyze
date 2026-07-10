---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 533
headings: []
---

# Source page 533

NVM Express® Base Specification, Revision 2.1

511
Figure 612: Data Replication Example 1

Figure 613 shows another example of single NVM subsystem host connectivity, where two hosts access
the same dispersed namespace being replicated across two participating NVM subsystems via controllers
in only one NVM subsystem, but the host connected to NVM Subsyste m 2 only accesses namespace B in
the event of a failure scenario.
Figure 613: Data Replication Example 2

Figure 614 shows an example of multiple NVM subsystem host connectivity, where two hosts access the
same dispersed namespace being replicated across two participating NVM subsystem via controllers in
both NVM subsystems.
NSID 1

NSID 2

NVMe Controller(s)
NS A

NSID 1

NSID 3

NVMe Controller(s)
NS C
NS B
NVM Subsystem 1
 NVM Subsystem 2
Host 1
 Host 2
NSID 1

NSID 2

NVMe Controller(s)
NS A

NSID 1

NSID 3

NVMe Controller(s)
NS C
NS B
NVM Subsystem 1
 NVM Subsystem 2
Host 1
 Host 2
