---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 200
headings: []
---

# Source page 200

NVM Express® Base Specification, Revision 2.1

178
Figure 151: Asynchronous Event Information – Notice
Value Definition
05h 1 LBA Status Information Alert: I/O Command Set specific definition.
06h
Endurance Group Event Aggregate Log Page Change: Indicates that event entries for one or
more Endurance Groups (refer to section 5.1.12.1.10) have been added to the Endurance Group
Event Aggregate log. To clear this event, the host issues a Get Log Page command with the
Retain Asynchronous Event bit cleared to ‘0’ for the Endurance Group Event Aggregate log.
07h
Reachability Group Change: The Reachability Group information (refer to section 5.1.12.1.25)
related to a Reachability Group that contains namespaces attached to this controller has changed
(e.g., a member was added to or deleted from a Reachability Group). The current Reachability
Group information for attached namespaces is indicated in the Reachability Groups log page. To
clear this event, the host issues a Get Log Page command (refer to section 5.1.12) with the Retain
Asynchronous Event bit cleared to ‘0’ for the Reachability Groups log.
A controller shall not send this event if an Attached Namespace Attribute Changed asynchronous
event or an Allocated Namespace Attribute Changed asynchronous event  is sent for the same
event, such as a change due to:
a) the attachment of a namespace (refer to section 5.1.20);
b) the deletion of a namespace (refer to section 5.1.21); or
c) the detachment of a namespace (refer to section 5.1.20).
08h
Reachability Association Change:  The Reachability Association information (refer to section
5.1.12.1.26) related to a Reachability Association that contains namespaces attached to this
controller has changed (e.g., a member was added to or deleted from a Reachability Association).
The current Reachability Association information for attached namespaces is i ndicated in the
Reachability Associations log page. To clear this event, the host issues a Get Log Page command
(refer to section 5.1.12) with the Retain Asynchronous Event bit cleared to ‘0’ for the Reachability
Associations log.
A controller shall not send this event if an Attached Namespace Attribute Changed asynchronous
event or an Allocated Namespace Attribute Changed asynchronous event  is sent for the same
event, such as a change due to:
a) the attachment of a namespace (refer to section 5.1.20);
b) the deletion of a namespace (refer to section 5.1.21); or
c) the detachment of a namespace (refer to section 5.1.20).
