---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 323
headings: []
---

# Source page 323

NVM Express® Base Specification, Revision 2.1

301
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
17
Reachability Groups Change Notices Support (RGCNS): If this bit is set
to ‘1’, then the controller supports the Reachability Groups Change Notices
event, and the Reachability Association Change Notices event. If this bit is
cleared to ‘0’, then the controller does not support the Reachability Groups
Change Notices event, nor the Reachability Association Change Notices
event.
16
Temperature Threshold Hysteresis Recovery (TTHR): If this bit is set to
‘1’, then the controller supports the Temperature Threshold Hysteresis
Recovery event. If this bit is cleared to ‘0’, then the controller does not
support the Temperature Threshold Hysteresis Recovery event.
No log page is associated with this event.
15
Normal NVM Subsystem Shutdown (NNSS): If this bit is set to ‘1’, then
the controller supports the Normal NVM Subsystem Shutdown event. If this
bit is cleared to ‘0’, then the controller does not support the Normal NVM
Subsystem Shutdown event.
No log page is associated with this event.
14
Endurance Group Event Aggregate Log Page Change Notices
(EGEAN): If this bit is set to ‘1’, then the controller supports the Endurance
Group Event Aggregate Log Page Change Notices event. If this bit is
cleared to ‘0’, then the controller does not support the Endurance Group
Event Aggregate Log Page Change Notices event.
13
LBA Status Information Alert Notices (LSIAN): If this bit is set to ‘1’, then
the controller supports the LBA Status Information Alert Notices event
(refer to the NVM Command Set Specification). If this bit is cleared to ‘0’,
then the controller does not support the LBA Status Information Alert
Notices event.
12
Predictable Latency Event Aggregate Log Change Notices  (PLEAN):
If this bit is set to ‘1’, then controller supports the Predictable Latency Event
Aggregate Log Change Notices event. If this bit is cleared to ‘0’, then the
controller does not support the Predictable Latency Event Aggregate Log
Change Notices event.
11
Asymmetric Namespace Access Change Notices  (ANACN): If this bit
is set to ‘1’, then the controller supports sending Asymmetric Namespace
Access Change Notices. If this bit is cleared to ‘0’, then the controller does
not support the Asymmetric Namespace Access Change Notices event.
10 Reserved
9
Firmware Activation Notices  (FAN): If this bit is set to ‘1’, then the
controller supports the Firmware Activation Notices event. If this bit is
cleared to ‘0’, then the controller does not support the Firmware Activation
Notices event.
8
Attached Namespace Attribute Notices  (NSAN): If this bit is set to ‘1’,
then the controller supports the Attached Namespace Attribute Notices
event and the associated Changed Attached Namespace List log page. If
this bit is cleared to ‘0’, then the controller does not support the  Attached
Namespace Attribute Notices event nor the associated Changed Attached
Namespace List log page.
7:0 Reserved

99:96 M M O4
Controller Attributes (CTRATT): This field indicates attributes of the controller.
Bits Description
31:20 Reserved
