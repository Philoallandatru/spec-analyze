---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 145
headings: ["3.8.3 Capacity Reporting"]
---

# Source page 145

NVM Express® Base Specification, Revision 2.1

123
Figure 88: Horizontally-Organized Dual NAND NVM Subsystem
Endurance Group 2 (QLC)
Endurance Group 1 (SLC)
NVM Set 2
NVM Set 1
NVM Subsystem (SSD)
Port
Controller

The Capacity Configuration Descriptor  for this example  contains two Endurance Group Configuration
Descriptors. The first Endurance Group Configuration Descriptor for this example:
• indicates a Capacity Adjustment Factor of approximately 400;
• contains one NVM Set Identifier; and
• contains four Channel Configuration Descriptors. Each Channel Configuration Descriptor contains
one Media Unit Configuration Descriptor.
The second Endurance Group Configuration Descriptor for this example:
• indicates a Capacity Adjustment Factor of 100;
• contains one NVM Set Identifier; and
• contains four Channel Configuration Descriptors. Each Channel Configuration Descriptor contains
three Media Unit Configuration Descriptors.
 Capacity Reporting
For an NVM subsystem that does not support multiple domains, the capacity information reported in the
Identify Controller data structure (i.e., the TNVMCAP field and the UNVMCAP field in Figure 312) describes
the capacity for the NVM subsystem. If the MEGCAP field is non-zero, that field indicates the largest entity
(e.g., Endurance Group, NVM Set, namespace that contains formatted storage) that may be created in the
NVM subsystem.
For an NVM subsystem that supports multiple domains, the capacity information reported in the Identify
Controller data structure describes the capacity accessible by the controller processing the Identify
command. The host may use the Identify command to a ccess the Domain List data structure (refer to
section 5.1.13.2.15) to determine the domains that are accessible by the controller and the capacity
information for each of those domains. If the Max Endurance Group Domain Capacity field is non -zero,
then the field describes the largest entity (e.g., Endurance Group, names pace that contains formatted
storage) that may be created by this controller in the domain described by that Domain Attributes Entry.
For an NVM subsystem that supports Endurance Groups (refer to section 3.2.3), the host may use the
Identify command to access the Endurance Group List data structure (refer to section 5.1.13.2.16) to
determine the Endurance Groups that are accessible by the controller . To determine the capacity
