---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 299
headings: []
---

# Source page 299

NVM Express® Base Specification, Revision 2.1

277
Figure 287: Command Dword 10 – Log Specific Field
Bits Description
14:09 Reserved
08
FDP Event Type (FDPET): This bit specifies the type of events to be reported in the log page. If
this bit is set to ‘1’, then the controller shall report host events. If this bit is cleared to ‘0’, then the
controller shall report controller events.

Figure 288: FDP Events Log Page
Bytes Description
Header
03:00
Number of FDP Events (NUMFDPE): This field indicates the number of FDP events (refer
to Figure 289) contained in this log.
If Flexible Data Placement transitions from disabled to enabled in an Endurance Group, then
this field shall be cleared to 0h for that Endurance Group.
63:04 Reserved
FDP Event List
127:64 FDP Event 1: The oldest FDP event, if any.
… …
(NUMFDPC*64)+63:
(NUMFDPC*64)
FDP Event NUMFDPC: The most recent FDP event, if any.
The FDP events shall be listed in ascending order of occurrence. Because the Timestamp feature may or
may not have been set by the host after each Controller Level Reset and because the Event Timestamp
field is the time at which the event was created, the FDP events may or may not be in order of ascending
value of the Event Timestamp field.
The number of FDP events reported in the log page is equal to the value of the Number of FDP Events
field. The values of the bytes in this log page which follow the FDP Event List is implementation specific.
The format of the FDP event is described in Figure 289.
Figure 289: FDP Event
Bytes Description
00
Event Type (ETYP): This field indicates the type of event:
Value O/M1 ETS2 Definition Reference
Host Events
0h O No Reclaim Unit Not Fully Written To Capacity 5.1.12.1.31.1.1
1h O No Reclaim Unit Time Limit Exceeded 5.1.12.1.31.1.2
2h O No Controller Level Reset Modified Reclaim Unit
Handles 5.1.12.1.31.1.3
3h O No Invalid Placement Identifier 5.1.12.1.31.1.4
4h to 6Fh  Reserved
70h to 7Fh O  Vendor Specific Host Events
Controller Events
80h O Yes Media Reallocated 5.1.12.1.31.2.1
81h O No Implicitly Modified Reclaim Unit Handle 5.1.12.1.31.2.2
82h to EFh  Reserved
F0h to FFh O  Vendor Specific Controller Events
Notes:
1. O/M definition: O = Optional, M = Mandatory.
2. Event Type Specific field: Yes = the field is utilized, No = the field is not utilized.
