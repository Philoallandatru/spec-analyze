---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 104
headings: ["3.2.5 Domains and Divisions", "3.2.5.1 Overview"]
---

# Source page 104

NVM Express® Base Specification, Revision 2.1

82
Figure 70: Flexible Data Placement Logical View of Non-Volatile Storage

 Domains and Divisions
 Overview
An NVM subsystem may be made up of a single domain or multiple domains (i.e., two or more). A domain
is the smallest indivisible unit that shares state (e.g., power state, capacity information). An NVM subsystem
that supports multiple domains shall support Asymme tric Namespace Access Reporting (refer to section
8.1.1).
A common example of a simple implementation of an NVM subsystem is one that consists of a single
domain (i.e., multiple domains are not supported).
Each domain is independent, and the boundaries between domains are communication boundaries (e.g.,
fault boundaries, management boundaries). If multiple domains are present in an NVM subsystem, then
those domains cooperate in the operation of that NVM subsystem. If a domain is unable to cooperate in the
operation of the NVM subsystem, then the NVM subsystem has become divided.
A division is an event (e.g., failure of a domain) or action (e.g., management action or reconfiguration) within
the NVM subsystem that affects communication between the domains contained in the NVM subsystem
(refer to Figure 71 and Figure 72). If a division exists, global state within the NVM subsystem may be
impacted (e.g., a controller may only have information about the state of the domains with which the
controller is able to communicate). A division event or action may:
• affect access to namespaces (refer to section 8.1.1); or
• impact operations that have NVM subsystem scope (e.g., TNVMCAP, sanitize, format, SMART
information).
Endurance Group
Reclaim
Group 1
Reclaim
Unit
Reclaim
Unit
...
Reclaim
Group 0
Reclaim
Unit
Reclaim
Unit
...
Reclaim
Group 2
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
...
Reclaim
Group P-1
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
...
...
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
Reclaim
Unit
Reclaim Unit Handle 0
...
Reclaim Unit Handle 1
 Reclaim Unit Handle N-1
