---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 129
headings: []
---

# Source page 129

NVM Express® Base Specification, Revision 2.1

107
b) all namespaces attached to the controller and all media required to process Admin
commands shall be ready (i.e., commands are not permitted to be aborted with a status
code of Namespace Not Ready with the Do Not Retry bit cleared to ‘0’ or Admin Command
Media Not Ready with the Do Not Retry bit cleared to ‘0’).
• Controller Ready Independent of Media: After the controller is enabled, all namespaces attached
to the controller and media required to process Admin commands may or may not become ready
by the time the controller becomes ready. Any Admin command or I/O command that specifies one
or more namespaces attached to the controller is permitted to be aborted with a status code of
Namespace Not Ready with the Do Not Retry bit cleared to ‘0’ until CRTO.CRWMT amount of time
after the controller is enabled.
Admin commands that require access to the media are permitted to be aborted with a status code
of Admin Command Media Not Ready with the Do Not Retry bit cleared to ‘0’ until CRTO.CRWMT
amount of time after the controller is enabled. Refer to Figure 84 for a list of Admin commands that
are permitted to be aborted with a status code of Admin Command Media Not Ready.
 The controller shall be able to process without error as described in section 3.5.4.1:
a) all Admin commands not listed in Figure 84 by the time the controller is ready;
b) all Admin commands listed in Figure 84 no later than CRTO.CRWMT amount of time after
the controller is enabled; and
c) all I/O commands no later than  CRTO.CRWMT amount of time after the controller is
enabled.
Figure 84: Admin Commands Permitted to Return a Status Code of Admin Command Media Not
Ready
Admin Command Additional Restrictions
Capacity Management
Device Self-test
If the Device Self-Test would result in testing one or more namespaces, then returning
a status code of Admin Command Media Not Ready is permitted. If the Device Self -
Test would not result in testing any namespaces, then returning a status code of Admin
Command Media Not Ready is not permitted.
Firmware Commit
Firmware Image Download
Get LBA Status
Get Log Page
Get Log Page is only permitted to return a status code of Admin Command Media Not
Ready for the following log pages:
• Device Self-test
• Firmware Slot Information
• Telemetry Controller-Initiated
• Telemetry Host-Initiated
• Predictable Latency Per NVM Set
• Predictable Latency Event Aggregate
• Persistent Event Log
• LBA Status Information
• Endurance Group Event Aggregate
• Media Unit Status
• Supported Capacity Configuration List
• Boot Partition
• Reservation Notification
• Rotational Media Information
• Vendor Specific
Namespace Attachment
Namespace Management
Format NVM
Sanitize
