---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 539
headings: []
---

# Source page 539

NVM Express® Base Specification, Revision 2.1

517
is not allowed to be associated with more than one Placement Handle per namespace. There are no other
requirements on which Reclaim Unit Handle is associated with which Placement Handle.
Namespaces are allowed to utilize the same (i.e., share) Reclaim Unit Handle but may have restrictions
defined in the applicable I/O Command Set specification.
The host is required to enable the Data Placement Directive (refer to section  8.1.8.4) to submit write
commands that specify a Placement Identifier (refer to Figure 282 and Figure 283) in the DSPEC field. That
Placement Identifier uniquely specifies the Reclaim Unit in which to place the user data (refer to Figure
616). The controller determines the Reclaim Unit by using the Reclaim Unit Handle associated with the
Placement Handle that was specified by the host when the namespace was created. For example, if the
write command in Figure 616 specifies a Reclaim Group Identifier value of 2h and a Placement Handle
value of 1h, then the host is targeting the user data for that write command to be placed into the Reclaim
Unit associated with Reclaim Group 2h and Reclaim Unit Handle NRUH-1.
A Placement Identifier is invalid if:
• the Reclaim Group Identifier value is greater than or equal to the number of Reclaim Groups
supported by the enabled FDP configuration (refer to the NRG field in Figure 280); or
• the Placement Handle is greater than or equal to the number of Placement Handles supported by
the Namespace Management command of the applicable I/O Command Set specification.
If the Placement Identifier in the DSPEC field of a write command is invalid, then the controller shall:
• select the Reclaim Group and Reclaim Unit Handle accessible by the specified namespace to
determine the placement of the user data for that write command; and
• generate an Invalid Placement Identifier FDP Event if that event is enabled for the selected Reclaim
Unit Handle. To receive the FDP event, the host should enable the Invalid Placement Identifier FDP
Event on all Reclaim Unit Handles.
Any write command to a namespace that exists in an Endurance Group with Flexible Data Placement
enabled that does not specify the Data Placement Directive uses the Placement Handle value 0h and the
controller selects the Reclaim Group for that command. For example, if the write command does not specify
the Data Placement Directive, then, as illustrated in Figure 616, Placement Handle value of 0h is associated
with Reclaim Unit Handle 1h and the controller may place the user data into the Reclaim Unit associated
any Reclaim Group using Reclaim Unit Handle 1h.
