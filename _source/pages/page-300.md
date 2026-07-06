---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 300
headings: ["5.1.12.1.31.1 Host Events", "5.1.12.1.31.1.1 Reclaim Unit Not Fully Written To Capacity (Event Type 0h)", "5.1.12.1.31.1.2 Reclaim Unit Time Limit Exceeded (Event Type 1h)"]
---

# Source page 300

NVM Express® Base Specification, Revision 2.1

278
Figure 289: FDP Event
Bytes Description
01
FDP Event Flags (FDPEF):
Bits Description
07:03 Reserved
02
Location Valid (LV):  If this bit is set to ‘1’, then the Reclaim Group Identifier field and the
Reclaim Unit Handle Identifier field contain a reported value. If this bit is cleared to ‘0’, then the
Reclaim Group Identifier field and the Reclaim Unit Handle Identifier field do not contain a
reported value.
01 NSID Valid (NSIDV): If this bit is set to ‘1’, then the NSID field contains a reported value. If this
bit is cleared to ‘0’, then the NSID field does not contain a reported value.
00
Placement Identifier Valid (PIV):  If this bit is set to ‘1’, then the Placement Identifier field
contains a reported value. If this bit is cleared to ‘0’, then the Placement Identifier field does not
contain a reported value.

03:02
Placement Identifier (PID):  This field indicates the Placement Identifier associated with the FDP event.
This field is reserved if the PIV bit is cleared to ‘0’.
If the FDP event is an Invalid Placement Identifier event (i.e., Event Type 03h), then this field is Placement
Identifier from the write command submitted by the host.
11:04
Event Timestamp (ETMSP): This field indicates a timestamp indicating the current value of the Timestamp
feature (refer to section 5.1.25.1.7) when this FDP event occurred. The format of this field is as defined in
Figure 396).
15:12
Namespace Identifier (NSID): This field indicates the identifier of the namespace associated with the FDP
event. If the NSIDV bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should ignore this
field.
31:16 Event Type Specific (ETSP): Each Event Type indicates the use of this field and the format of this field.
33:32
Reclaim Group Identifier  (RGICD): This field indicates the Reclaim Group associated with the event. If
the LV bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should ignore this field.
If the FDP event is an Invalid Placement Identifier event (i.e., Event Type 03h), then this field indicates the
Reclaim Group selected by the controller.
35:34
Reclaim Unit Handle Identifier (RUHID): This field indicates the Reclaim Unit Handle associated with the
FDP event. If the LV bit is cleared to ‘0’, then this field shall be cleared to 0h and the host should ignore this
field.
If the FDP event is an Invalid Placement Identifier event (i.e., Event Type 03h), then this field indicates the
Reclaim Unit Handle selected by the controller.
39:36 Reserved
Vendor Specific Information
63:40 Vendor Specific (VS): This field may be used by any Event Type (i.e., this field is not only for Vendor
specific Event Types).
If the controller is not able to report the namespace identifier associated with the event, then the NSIDV bit
shall be cleared to ‘0’.
If the controller is not able to report the specific Reclaim Unit Handle, then:
• the PIV bit shall be cleared to ‘0’; and
• the LV bit shall be cleared to ‘0’.
 Host Events
5.1.12.1.31.1.1 Reclaim Unit Not Fully Written To Capacity (Event Type 0h)
This FDP event indicates that the Reclaim Unit Handle reported in the Placement Identifier (refer to Figure
282 and Figure 283) was modified to reference a different Reclaim Unit due to an I/O Management Send
command issued by the host prior to the previously referenced Reclaim Unit being written to capacity.
5.1.12.1.31.1.2 Reclaim Unit Time Limit Exceeded (Event Type 1h)
