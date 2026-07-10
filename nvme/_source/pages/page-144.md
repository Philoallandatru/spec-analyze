---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 144
headings: ["3.8.2.3 Horizontally-Organized Dual NAND NVM Subsystem"]
---

# Source page 144

NVM Express® Base Specification, Revision 2.1

122
Figure 87: Vertically-Organized NVM Subsystem

The Capacity Configuration Descriptor for this example contains four Endurance Group Configuration
Descriptors. Each Endurance Group Configuration Descriptor contains one NVM Set Identifier and one
Channel Configuration Descriptor. Each Channel Configurati on Descriptor contains four Media Unit
Configuration Descriptors.
 Horizontally-Organized Dual NAND NVM Subsystem
Figure 88 shows an example of a single domain NVM subsystem where  the Media Units are NAND which
is capable of being operated as QLC or at a lower density. The performance goal is for maximum bandwidth
to a small NVM Set operating as SLC and to a larger NVM Set operating as QLC.
