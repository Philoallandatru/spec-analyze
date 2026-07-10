---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 201
headings: []
---

# Source page 201

NVM Express® Base Specification, Revision 2.1

179
Figure 151: Asynchronous Event Information – Notice
Value Definition
09
Allocated Namespace Attribute Changed: Indicates a change to one or more of the following:
• Identify Namespace data structures (refer to section 1.5.49) that are associated with an
allocated namespace (refer to section 1.5.6);
• the Active Namespace ID list (i.e., CNS 02h);
• the I/O Command Set specific Active Namespace ID list (i.e., CNS 07h);
• the Allocated Namespace ID list (i.e., CNS 10h);
• the I/O Command Set specific Allocated Namespace ID list (i.e., CNS 1Ah); or
• other data structures as specified in the applicable NVMe I/O Command Set
specifications.
This event shall be reported for any of the changes in the preceding list to namespaces that are
either attached to the controller or not attached to the controller, unless otherwise specified.
To clear this event, a host issues a Get Log Page command for the Changed Allocated
Namespace List log page (refer to section 5.1.12.1.27) with the Retain Asynchronous Event bit
cleared to ‘0’.
A controller shall not send this event if:
a) Namespace Status (refer to Figure 319) has changed and shutdown processing is either
occurring (i.e., CSTS.SHST is set to 01b) or complete (i.e., CSTS.SHST is set to 10b);
b) that controller receives a command (e.g., a Namespace Management command or
Capacity Management command) on the Admin Submission Queue (e.g., not via a
Management Endpoint, refer to the NVM Express Management Interface Specification)
that requests a delete operation for a namespace;
c) the ANAGRPID field (refer to Figure 319) has changed;
d) the RGRPID field (refer to Figure 319) has changed; or
e) an I/O Command Set specific change occurs (refer to the applicable I/O Command Set
specification).
A controller shall only send this event for changes to the Format Progress Indicator (FPI) field
when the Remaining Format NVM field of that FPI field transition from a non-zero value to 0h, or
from 0h to a non-zero value.
0Ah to EEh Reserved
EFh 2 Zone Descriptor Changed: I/O Command Set specific definition.
F0h
Discovery Log Page Change:  A change has occurred to one or more of the Discovery log
pages. To clear this event, the host or Discovery controller reads the Discovery log page with the
Retain Asynchronous Event bit cleared to ‘0’.
F1h
Host Discovery Log Page Change:  A change has occurred to one or more of the Host
Discovery log pages. The host or Discovery controller should submit a Get Log Page command
to receive updated Host Discovery log pages.
F2h
AVE Discovery Log Page Change: A change has occurred to the AVE Discovery log page. The
host or controller should submit a Get Log Page command to receive an updated AVE Discovery
log page.
F3h Pull Model DDC Request: A pull model DDC is requesting a CDC to perform an operation.
F4h to FFh Reserved for future NVMe over Fabrics Asynchronous Event Notifications
Notes:
1. Refer to the NVM Command Set Specification.
2. Refer to the Zoned Namespace Command Set Specification.

Figure 152: Asynchronous Event Information – I/O Command Specific Status
Value Definition
00h
Reservation Log Page Available: Indicates that one or more Reservation Notification log pages
(refer to section 5.1.12.1.32) have been added to the Reservation Notification log . To clear this
event, the host reads the Reservation log page with the Retain Asynchronous Event bit cleared
to ‘0’.
