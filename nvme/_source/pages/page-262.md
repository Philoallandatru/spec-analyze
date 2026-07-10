---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 262
headings: ["5.1.12.1.14.2.5 NVM Subsystem Hardware Error Event (Event Type 05h)"]
---

# Source page 262

NVM Express® Base Specification, Revision 2.1

240
A Power-on or Reset event shall be recorded in the Persistent Event Log when an NVM Subsystem Reset
(e.g., due to a power -on) or a Controller Level Reset is completed. The Power -on or Reset Event reports
information about resets due to power -on or other events that cause resets (refer to section  3.7) followed
by descriptors reporting information about the controller at the time the reset occurred, including timestamp
values for all controllers for use in synchronization of timestamp values across controllers.
The controller shall set the Persistent Event Log Event Format Header:
a) Event Type field to 04h; and
b) Event Type Revision field to 01h.
The Power-on or Reset Event data is specified in Figure 233.
Figure 233: Power-on or Reset Event (Event Type 04h)
Bytes Description
07:00 Firmware Revision (FREV): Contains the firmware revision that becomes effective when CC.EN
transitions from ‘0’ to ‘1’.
EL-VSIL-1:081
Reset Information List  (RIL): Contains a list of one or more Controller Reset Information
descriptors (refer to Figure 234). If virtualization management is not implemented, then the list
shall contain a Controller Reset Information descriptor for every controller in the NVM subsystem.
If virtualization management is implemented, then the list shall contain a Controller Reset
Information descriptor for every primary controller.
The Controller Reset Information descriptor is shown in Figure 234.
Notes:
1. Refer to Figure 227 for the definitions of EL and VSIL.

Figure 234: Controller Reset Information descriptor
Bytes Description
01:00 Controller ID (CNTLID): Contains the Controller ID for the controller with the timestamp in the Controller
Timestamp field.
02
Firmware Activation (FA): Contains a code indicating if this event triggered a firmware activation.
Value Definition
00h Indicates that this event did not trigger a firmware activation on the controller.
01h Indicates that new firmware was activated on the controller due to this power on or reset.
02h Indicates that an attempt to activate new firmware on the controller due to this power-on or
reset failed.
03h to
FFh Reserved

03
Operation in Progress (OIP):
Bits Description
7:1 Reserved
0
Reset During NVM Format (RDNF): If this bit is set to ‘1’, then a Format NVM command was in
progress for a namespace attached to the controller when this reset event occurred. If this bit is
cleared to ‘0’, then  no Format NVM command was in progress for any namespace attached to
the controller when this reset event occurred.

15:04 Reserved
19:16 Controller Power Cycle  (CPWRC): Contains the power cycle count for the controller indicated in the
Controller ID field.
27:20
Power on milliseconds (POM): Contains the power on hours in milliseconds since being manufactured.
This may not include time that the controller was powered and in a non-operational power state.
The resolution of this field is vendor specific (e.g., an NVM subsystem that only counts power on time in
hours only reports values corresponding to whole hours).
35:28 Controller Timestamp (CTSTP): Contains a timestamp for the controller specified in the Controller ID field
at the time when this event occurred using the Timestamp data structure defined in Figure 395.
5.1.12.1.14.2.5 NVM Subsystem Hardware Error Event (Event Type 05h)
