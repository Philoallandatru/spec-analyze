---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 143
headings: ["3.8.2.2 Vertically-Organized NVM Subsystem"]
---

# Source page 143

NVM Express® Base Specification, Revision 2.1

121
Figure 86: Simple NVM Subsystem
Endurance Group 1
NVM Subsystem (SSD)
Port
NVM Set 1
Controller
Key:
MU
Channel
NVM Set
Endurance
Group
Set 1
EG 1

The Capacity Configuration Descriptor for this example contains one Endurance Group Configuration
Descriptor. The Endurance Group Configuration Descriptor contains one NVM Set Identifier and four
Channel Configuration Descriptors. Each Channel Configuration Descriptor contains four Media Unit
Configuration Descriptors.
 Vertically-Organized NVM Subsystem
Figure 87 shows an example of a si ngle domain NVM subsystem where the performance goal is isolation
among four NVM Sets at the cost of bandwidth. Endurance is managed separately for each set. Media
Units sharing a Channel are allocated to the same Endurance Group. All Media Units in an Endurance
Group are allocated to the same NVM Set. The bandwidth for any NVM Set is likely to be less than or equal
to the bandwidth of the Channel of that NVM Set.
