---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 239
headings: []
---

# Source page 239

NVM Express® Base Specification, Revision 2.1

217
operation and so on. If fewer than 20 self-test operations have completed or been aborted, then the Device
Self-test Result field shall be set to Fh (i.e., Entry is empty) and the Self-test Code field shall be cleared to
0h (i.e., Entry is empty ) in the Device Self-test Status field in the unused Self -test Result Data Structure
fields and all other fields in that Self-test Result data structure should be ignored by the host.
Figure 211: Device Self-test Log Page
Bytes Description
00
Current Device Self -Test Operation (CDSTO): This field defines the current device self -test
operation.
Bits Description
7:4 Reserved
3:0
Device Self-test Operation Status (DSTOS): This field indicates the status of the
current device self-test operation as defined in the following table. If a device self-
test operation is in process (i.e., this field is set to 1h or 2h), then the controller shall
not set this field to 0h until a new Self -test Result Data Structure is created (i.e., if
a device self-test operation completes or is aborted, then the controller shall create
a Self-test Result Data Structure prior to setting this field to 0h).
Value Definition
0h No device self-test operation in progress
1h Short device self-test operation in progress
2h Extended device self-test operation in progress
3h to Dh Reserved
Eh Vendor specific
Fh Reserved


01
Current Device Self-Test Completion (CDSTC): This field defines the completion status of the
current device self-test.
Bits Description
7 Reserved
6:0
Device Self-test Completion Status (DSTCS): This field indicates the percentage
of the device self-test operation that is complete (e.g., a value of 25 indicates that
25% of the device self-test operation is complete and 75% remains to be tested). If
the Device Self-test Operation Status (DSTOS) field in the Current Device Self-Test
Operation field is cleared to 0h (i.e., there is no device se lf-test operation in
progress), then this field should be ignored by the host.

03:02 Reserved
Device Self-test Result List
31:04 Result Data Structure 1 (RDS1):  The newest Self-test Result data structure (refer to Figure
212)
59:32 Result Data Structure 2 (RDS2): The 2nd newest Self-test Result data structure (refer to Figure
212)
… …
535:508 Result Data Structure 19 (RDS19):  The 19th newest Self -test Result data structure (refer to
Figure 212)
563:536 Result Data Structure 20 (RDS20):  The 20th newest Self -test Result data structure (refer to
Figure 212)
