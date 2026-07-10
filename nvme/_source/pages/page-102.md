---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 102
headings: ["3.2.3.1 Configuring and Managing Endurance Group Events"]
---

# Source page 102

NVM Express® Base Specification, Revision 2.1

80
Figure 69: NVM Sets and Associated Namespaces

If Endurance Groups are supported, then the NVM subsystem and all controllers shall:
• indicate support for Endurance Groups in the Controller Attributes field in the Identify Controller
data structure;
• indicate the Endurance Group Identifier with which the namespace is associated in the Identify
Namespace data structure;
• support the Endurance Group Information log page; and
• support the Endurance Group Event Aggregate log page  if more than one Endurance Group is
supported in the NVM subsystem.
If Endurance Groups are not supported and the host sends a command in which an Endurance Group
Identifier field is defined (e.g., Get Log Page), then that field shall be ignored by the controller.
If Endurance Groups are not supported  and the controller returns information to the host that contains an
Endurance Group Identifier field, then that field shall be cleared to 0h.
 Configuring and Managing Endurance Group Events
The host may configure asynchronous events to be triggered when certain events occur for an Endurance
Group. The host submits a Set Features command specifying the Endurance Group Event Configuration
feature (refer to section 5.1.25.1.16), the Endurance Group, and the specific event(s) that shall trigger
adding an entry to the Endurance Group Event Aggregate log page (refer to section 5.1.12.1.15).
The host configures events using a Set Features command for each Endurance Group.
The host submits a Set Features command specifying the Asynchronous Event Configuration feature (refer
to section 5.1.25.1.5) with the Endurance Group Event Aggregate Log Change Notices bit set to ‘1’ to
specify that adding an entry to the Endurance Group Event Aggregate log page shall trigger an Endurance
Group Event Aggregate Log Page Change Notice event to the host (refer to Figure 409).
