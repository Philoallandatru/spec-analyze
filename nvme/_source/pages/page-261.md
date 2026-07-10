---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 261
headings: ["5.1.12.1.14.2.2 Firmware Commit Event (Event Type 02h)", "5.1.12.1.14.2.3 Timestamp Change Event (Event Type 03h)", "5.1.12.1.14.2.4 Power-on or Reset Event (Event Type 04h)"]
---

# Source page 261

NVM Express® Base Specification, Revision 2.1

239
The SMART / Health Log Snapshot Event shall set the Persistent Event Log Event Header:
a) Event Type field to 01h; and
b) Event Type Revision field to 01h.
The SMART / Health Log Snapshot Event data is specified in Figure 230.
Figure 230: SMART / Health Log Snapshot Event Data Format (Event Type 01h)
Bytes Description
511:00 Event Data (ED): Contains a snapshot of the SMART/Health Information Log data specified in Figure
206.
5.1.12.1.14.2.2 Firmware Commit Event (Event Type 02h)
A firmware commit event shall be recorded in the Persistent Event Log when a Firmware Commit command
is completed. The Firmware Commit Event shall set the Persistent Event Log Event Format Header:
a) Event Type field to 02h; and
b) Event Type Revision field to 01h.
The Firmware Commit Event data is specified in Figure 231.
Figure 231: Firmware Commit Event Data Format (Event Type 02h)
Bytes Description
07:00 Old Firmware Revision  (OFR): Contains the firmware revision of the active firmware before this
firmware commit event.
15:08 New Firmware Revision (NFR): Contains the firmware revision for the firmware that was requested to
become active.
16 Firmware Commit Action  (FCA): Contains the value from the Commit Action field in the Firmware
Commit command.
17 Firmware Slot  (FSLT): Contains the value from the Firmware Slot field in the Firmware Commit
command.
18 Status Code Type for Firmware Commit Command (STCTFCC): Contains the status code type from
the completion queue entry for the Firmware Commit command.
19 Status Returned for Firmware Commit Command  (SRFCC): Contains the status code from the
completion queue entry for the Firmware Commit command.
21:20
Vendor Assigned Firmware Commit Result Code  (VAFCRC): Contains a vendor specific value that
provides more information about the result of the firmware commit. A value of 0h indicates that no vendor
assigned firmware commit result code is provided.
5.1.12.1.14.2.3 Timestamp Change Event (Event Type 03h)
The Timestamp Change Event (refer to Figure 232) contains the current timestamp, reported in the event
header, and the timestamp from the time at which the timestamp was changed (i.e., the old timestamp).
The Timestamp Change Event shall set the Persistent Event Log Event Format Header:
a) Event Type field to 03h; and
b) Event Type Revision field to 01h.
The Timestamp Change Event data is specified in Figure 232.
Figure 232: Timestamp Change Event Format (Event Type 03h)
Bytes Description
07:00 Previous Timestamp (PTSTP): Contains a timestamp of the time immediately before the timestamp
was changed (i.e., the old timestamp) using the Timestamp data structure as defined in Figure 395.
15:08 Milliseconds Since Reset (MSR): Contains the time since the last Controller Level Reset reported in
milliseconds.
5.1.12.1.14.2.4 Power-on or Reset Event (Event Type 04h)
