---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 400
headings: ["5.1.25.1.6 Autonomous Power State Transition (Feature Identifier 0Ch)"]
---

# Source page 400

NVM Express® Base Specification, Revision 2.1

378
Figure 391: Asynchronous Event Configuration – Command Dword 11
Bits Description
15
Normal NVM Subsystem Shutdown  (NNSSHDN): This bit determines whether an asynchronous
event notification is sent to the host when the NVM subsystem has started performing a normal
shutdown due to an NVM Subsystem Shutdown (refer to  Figure 153). If this bit is set to ‘1’, then the
Normal NVM Subsystem Shutdown event is sent to the host if an outstanding Asynchronous Event
Request command exists at the time this condition occurs. If this bit is cleared to ‘0’, then the controller
shall not send the Normal NVM Subsystem Shutdown event to the host.
14
Endurance Group Event Aggregate Log Change Notices (EGEALCN): This bit determines whether
an asynchronous event notification is sent to the host when an event entry for an Endurance Group
(refer to section 3.2.3) has been added to the Endurance Group Event Aggregate log (refer to section
5.1.12.1.15). If this bit is set to ‘1’, then the Endurance Group Event Aggregate Log Change event is
sent to the host when this condition occurs. If this bit is cleared to ‘0’, then the controller shall not send
the Endurance Group Event Aggregate Log Change event to the host.
If Endurance Groups are not supported and this bit is set to ‘1’, then the Set Features command shall
be aborted with a status of Invalid Field in Command.
13 LBA Status Information Alert Notices1 (LSIAN): I/O Command Set specific definition.
12
Predictable Latency Event Aggregate Log Change Notices  (PLEALCN): This bit determines
whether an asynchronous event notification is sent to the host when an event pending entry for an
NVM Set (refer to section 5.1.12.1.12) has been added to the Predictable Latency Event Aggregate
Log. If this bit is set to ‘1’, then the Predictable Latency Event Aggregate Log Change event is sent to
the host when this condition occurs. If this bit is cleared to ‘0’, then the controller sha ll not send the
Predictable Latency Event Aggregate Log Change event to the host.
11
Asymmetric Namespace Access Change Notices  (ANACN): This bit determines whether an
asynchronous event notification is sent to the host when an asymmetric namespace access change
occurs (i.e., the contents of the Asymmetric Namespace Access log page (refer to section 5.1.12.1.13)
change). If this bit is set to ‘1’, then the Asymmetric Namespace Access Change Notices event is sent
to the host when this condition occurs. If this bit is cleared to ‘0’, then the controller shall not send the
Asymmetric Namespace Access Change Notices event to the host.
10
Telemetry Log Notices (TLN): This bit determines whether an asynchronous event notification is sent
to the host when the Telemetry Controller-Initiated Data Available field transitions from 0h to 1h in the
Telemetry Controller-Initiated log page. If this bit is set to ‘1’, then the Telemetry Log Changed event
is sent to the host when this condition occurs. If this bit is cleared to ‘0’, then the controller shall not
send the Telemetry Log Changed event to the host.
09
Firmware Activation Notices (FAN): This bit determines whether an asynchronous event notification
is sent to the host for a Firmware Activation Starting event (refer to Figure 151). If this bit is set to ‘1’,
then the Firmware Activation Starting event is sent to the host when this condition occurs. If this bit is
cleared to ‘0’, then the controller shall not send the Firmware Activation Starting event to the host.
08
Attached Namespace Attribute Notices (NAN): This bit determines whether an asynchronous event
notification is sent to the host for a n Attached Namespace Attribute Changed asynchronous event
(refer to Figure 151). If this bit is set to ‘1’, then the Attached Namespace Attribute Changed
asynchronous event is sent to the host when this condition occurs. If this bit is cleared to ‘0’, then the
controller shall not send the Attached Namespace Attribute Changed asynchronous event to the host.
07:00
SMART / Health Critical Warnings  (SHCW): This field determines whether an asynchronous event
notification is sent to the host for the corresponding Critical Warning specified in the SMART / Health
Information log (refer to Figure 206). If a bit is set to ‘1’, then an asynchronous event notification is
sent when the corresponding critical warning bit is set to ‘1’ in the SMART / Health Information log. If
a bit is cleared to ‘0’, then an asynchronous event notification is not sent when  the corresponding
critical warning bit is set to ‘1’ in the SMART / Health Information log.
Notes:
1. Refer to the NVM Command Set Specification.
2. Refer to the Zoned Namespace Command Set Specification.
5.1.25.1.6 Autonomous Power State Transition (Feature Identifier 0Ch)
This Feature configures the settings for autonomous  controller power state transitions, refer to section
8.1.17.2.
