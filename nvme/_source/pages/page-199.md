---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 199
headings: []
---

# Source page 199

NVM Express® Base Specification, Revision 2.1

177
Figure 151: Asynchronous Event Information – Notice
Value Definition
00h
Attached Namespace Attribute Changed: Indicates a change to one or more of the following:
• Identify Namespace data structures (refer to section 1.5.49) that are associated with an
attached namespace;
• the Active Namespace ID list (i.e., CNS 02h);
• the I/O Command Set specific Active Namespace ID list (i.e., CNS 07h); or
• other data structures as specified in applicable NVMe I/O Command Set specifications.
This event shall be reported for any of the changes in the preceding list to namespaces that are
attached to the controller, unless otherwise specified. This event may be reported by controllers
compliant to previous versions of this specification for chan ges to namespaces that are not
attached to the controller.
To clear this event, a host issues a Get Log Page command for the Changed Attached
Namespace List log page (refer to section 5.1.12.1.5) with the Retain Asynchronous Event bit
cleared to ‘0’.
A controller shall not send this event if:
a) Namespace Status (refer to Figure 319) has changed and shutdown processing is either
reported as in progress (i.e., CSTS.SHST is set to 01b) or is reported as complete (i.e.,
CSTS.SHST is set to 10b);
b) that controller receives a command (e.g., a Namespace Management command or
Capacity Management command) on the Admin Submission Queue (e.g., not via a
Management Endpoint, refer to the NVM Express Management Interface Specification)
that requests a delete operation for a namespace;
c) the ANAGRPID field (refer to Figure 319) has changed;
d) the RGRPID field (refer to Figure 319) has changed; or
e) an I/O Command Set specific change occurs (refer to the applicable I/O Command Set
specification).
A controller shall only send this event for changes to the Remaining Format NVM (RFNVM) field
in the Format Progress Indicator field transition from a non-zero value to 0h, or from 0h to a non-
zero value.
01h
Firmware Activation Starting: The controller is starting a firmware activation process during
which command processing is paused. The host may use CSTS.PP to determine when command
processing has resumed.  To clear this event, the host reads the Firmware Slot Information log
page with the Retain Asynchronous Event bit cleared to ‘0’.
02h
Telemetry Log Changed: The controller has saved the controller internal state in the Telemetry
Controller-Initiated log page and set the Telemetry Controller -Initiated Data Available field to 1h
in that log page. To clear this event, the host issues a Get Log Page command with Retain
Asynchronous Event bit cleared to ‘0’ for the Telemetry Controller-Initiated log.
03h
Asymmetric Namespace Access Change:  The Asymmetric Namespace Access information
(refer to section 5.1.12.1.13) related to an ANA Group that contains namespaces attached to this
controller has changed  (e.g., an ANA state has changed, an ANAGRPID has changed) . The
current Asymmetric Namespace Access information for attached namespaces is indicated in the
Asymmetric Namespace Access log page (refer to section 5.1.12.1.13). To clear this event, the
host issues a Get Log Page  command (refer to section 5.1.12) with the Retain Asynchronous
Event bit cleared to ‘0’ for the Asymmetric Namespace Access log.
A controller shall not send this event if an Attached Namespace Attribute Changed asynchronous
event or an Allocated Namespace Attribute Changed asynchronous event  is sent for the same
event, such as a change due to:
a) the attachment of a namespace (refer to section 5.1.20);
b) the deletion of a namespace (refer to section 8.1.15.2); or
c) the detachment of a namespace (refer to section 5.1.20).
04h
Predictable Latency Event Aggregate Log Change:  Indicates that event pending entries for
one or more NVM Sets (refer to section 5.1.12.1.12) have been added to the Predictable Latency
Event Aggregate log. To clear this event, the host reads the Predictable Latency Event Aggregate
log page with the Retain Asynchronous Event bit cleared to ‘0’.
