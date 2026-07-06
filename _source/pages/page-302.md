---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 302
headings: ["5.1.12.1.33 Sanitize Status (Log Page Identifier 81h)"]
---

# Source page 302

NVM Express® Base Specification, Revision 2.1

280
Figure 290: Reservation Notification Log Page
Bytes Description
07:00
Log Page Count  (LPC): This is a 64 -bit incrementing Reservation Notification log page count,
indicating a unique identifier (modulo 64 bit) for this notification. The count starts at 0h following a
Controller Level R eset and is incremented for every event that causes a reservation notification
regardless of whether that notification is added to the queue . If t he value of this field is
FFFFFFFF_FFFFFFFFh, then the field is set to 1h when incremented (i.e., rolls over to 1h) and a new
log page is created.
If there are no Reservation Notification log pages to return (i.e., the queue of Reservation Notification
log pages is empty), then this field shall return the value 0h. Subsequent reservation notifications
continue incrementing this unique identifier from the last non -zero value (i.e., the value that identified
the previous Reservation Notification log page). A value of 0h indicates the log page is empty.
08
Reservation Notification Log Page Type  (RNLPT): This field indicates the Reservation Notification
type described by this log page.
Value Definition
0h
Empty Log Page:  Get Log Page command was processed when no unread
Reservation Notification log pages were available. All the fields of an empty log
page shall have a value of 0h.
1h Registration Preempted
2h Reservation Released
3h Reservation Preempted
4h to FFh Reserved

09
Number of Available Log Pages  (NALP): This field indicates the number of additional available
Reservation Notification log pages (i.e., the number of unread log pages not counting this one). If there
are more than 255 additional available log pages, then a value of 255 is returned. A value of 0h indicates
that there are no additional available log pages.
11:10 Reserved
15:12 Namespace ID (NSID): This field indicates the namespace ID of the namespace associated with the
Reservation Notification described by this log page.
63:16 Reserved
5.1.12.1.33 Sanitize Status (Log Page Identifier 81h)
The Sanitize Status log page is used to report sanitize operation time estimates and information about the
most recent sanitize operation (refer to section 8.1.24). The Get Log Page command returns a data buffer
containing a log page formatted as defined in Figure 291. This log page shall be retained across power
cycles and resets. This log page shall contain valid data whenever CSTS.RDY is set to ‘1’.
If the Sanitize Capabilities (SANICAP) field in the Identify Controller data structure is not cleared to 0h (i.e.,
the Sanitize command is supported), then this log page shall be supported. If the Sanitize Capabilities field
in the Identify Controller data structure is cleared to 0h, then this log page is reserved.
Figure 291: Sanitize Status Log Page
Bytes Description
01:00
Sanitize Progress (SPROG): This field indicates the fraction complete of the:
• sanitize processing state (i.e., the Restricted Processing state or the Unrestricted Processing
state); or
• Post-Verification Deallocation state, if the Post -Verification Deallocation state is entered as
part of the sanitize operation.
The value is the numerator of the fraction complete that has 65,536 (10000h) as its denominator. This
value shall be set to FFFFh if the Sanitize Operation Status (SOS) field is set to a value other than 010b
(i.e., Sanitizing) or if the sanitization target is in the Media Verification state. Refer to section 8.1.24.3
for the effects of state changes that change the value of this field.
