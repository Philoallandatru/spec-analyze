---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 399
headings: ["5.1.25.1.5 Asynchronous Event Configuration (Feature Identifier 0Bh)"]
---

# Source page 399

NVM Express® Base Specification, Revision 2.1

377
Figure 390: Volatile Write Cache – Command Dword 11
Bits Description
31:01 Reserved
00 Volatile Write Cache Enable (WCE): If this bit is set to ‘1’, then the volatile write cache is enabled.  If
this bit is cleared to ‘0’, then the volatile write cache is disabled.
5.1.25.1.5 Asynchronous Event Configuration (Feature Identifier 0Bh)
This Feature controls the events that trigger an asynchronous event notification to the host . This Feature
may be used to disable reporting events  by the controller  in the case of a persistent condition (refer to
section 5.1.2). If the condition for an event is true when the corresponding notice is enabled, then an event
is sent to the host. The attributes are specified in Command Dword 11.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 391 are returned
in Dword 0 of the completion queue entry for that command.
Figure 391: Asynchronous Event Configuration – Command Dword 11
Bits Description
Fabrics Specific
31
Discovery Log Page Change Notification (DLPCN): This bit indicates that the Discovery controller
reports Discovery Log Page Change Notifications. If this bit is set to ‘1’, then the Discovery controller
shall send a notification if Discovery log page changes occur.
30
Host Discovery Log Page Change Notification (HDLPCN): This bit indicates that the Discovery
controller reports Host Discovery Log Page Change Notifications. If this bit is set to ‘1’, then the
Discovery controller shall send a notification if Host Discovery log page changes occur.
29
AVE Discovery Log Page Change Notification  (ADLPCN): This bit indicates that the Discovery
controller reports AVE Discovery Log Page Change Notifications. If this bit is set to ‘1’, then the
Discovery controller shall send a notification if AVE Discovery log page changes occur.
28
Pull Model DDC Request Log Page Change Notification (PMDRLPCN): This bit indicates that the
Discovery controller reports Pull Model DDC Request log page Change Notifications. If this bit is set
to ‘1’, then the Discovery controller shall send a notification if Pull Model DDC Request log page
changes occur.
Non-Fabrics Specific
27 Zone Descriptor Changed Notices2 (ZDCN): I/O Command Set specific definition.
26:20 Reserved
19
Allocated Namespace Attribute Notices  (ANSAN): This bit determines whether an asynchronous
event notification is sent to the host for an Allocated Namespace Attribute Changed asynchronous
event (refer to Figure 151). If this bit is set to ‘1’, then the Allocated Namespace Attribute Changed
asynchronous event is sent to the host when this condition occurs. If this bit is cleared to ‘0’, then the
controller shall not send the Allocated Namespace Attribute Changed asynchronous event to the host.
18
Reachability Group (RGRP0): This bit determines whether an asynchronous event notification is sent
to the host when a Reachability Group change occurs (i.e., the contents of the Reachability Group s
log page (refer to section 5.1.12.1.25) changed). If this bit is set to ‘1’, then the Reachability Group
Change Notices event is sent to the host when this condition occurs. If this bit is cleared to ‘0’, then
the controller shall not send the Reachability Group Change Notices event to the host.
17
Reachability Association (RASSN): This bit determines whether an asynchronous event notification
is sent to the host when a Reachability Association change occurs (i.e., the contents of the Reachability
Associations log page (refer to section 5.1.12.1.26) change d). If this bit is set to ‘1’, then the
Reachability Association Change Notices event is sent to the host when this condition occurs. If this
bit is cleared to ‘0’, then the controller shall not send the Reachability Association Change Notices
event to the host.
16
Temperature Threshold Hysteresis Recovery  (TTHRY): This bit determines whether an
asynchronous event notification is sent to the host at the end of a temperature threshold hysteresis
event (refer to section 5.1.25.1.3.1). If this bit is set to ‘1’, then the Temperature Threshold Hysteresis
Recovery event is sent to the host if an outstanding Asynchronous Event Request command exists at
the time this condition occurs. If this bit is cleared to ‘0’, then the controller sha ll not send the
Temperature Threshold Hysteresis Recovery event to the host.
