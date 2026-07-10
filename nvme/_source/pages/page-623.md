---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 623
headings: []
---

# Source page 623

NVM Express® Base Specification, Revision 2.1

601
Figure 670: Controller Resource Allocation

For each controller resource type supported, the Primary Controller Capabilities Structure (refer to Figure
330) defines:
• The total number of Flexible Resources;
• The total number of Private Resources for the primary controller;
• The maximum number of Flexible Resources that may be assigned to a secondary controller using
the Virtualization Management command; and
• The assignment of resources to the primary controller.
Primary and secondary controllers may implement all features of this specification, except where
commands are defined as being only supported by a primary controller. It is recommended that only primary
controllers support the privileged actions described in section 3.10 so that untrusted hosts using secondary
controllers do not impact the entire NVM subsystem state.
The Secondary Controller List structure returned by the Identify command is used to determine the topology
of secondary controllers and the resources assigned. The secondary controller shall be in the Offline state
to configure resources. The Virtualizatio n Management command is used to transition the secondary
controller between the Online state and the Offline state. Refer to section 8.2.6.3 for details on the Online
and Offline states.
To support the Virtualization Enhancements capability, the NVM subsystem shall support the following:
• One or more primary controllers, each of which supports:
o One or more secondary controllers;
o A pool of unassigned Flexible Resources that supports allocation to a primary controller and
dynamic assignment to its associated secondary controllers;
o Two or more Private Resource queue pairs;
