---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 269
headings: ["5.1.12.1.14.2.9 Sanitize Start Event (Event Type 09h)", "5.1.12.1.14.2.10 Sanitize Completion Event (Event Type 0Ah)"]
---

# Source page 269

NVM Express® Base Specification, Revision 2.1

247
Figure 242: Format NVM Completion Event Data Format (Event Type 08h)
Bytes Description
07:06
Completion Information (CINFO): Contains a vendor specific value that may provide more information
about the completion of the format operation (e.g., if the format operation did not complete successfully,
then this field may contain a vendor specific code that indicates a vendor specific reason).
09:08
Status Info (INFO): Identifies the status for the NVM Format command.
Bits Description
15:1
Status (STATUS): This field indicates the value that was reported in the Status field for the
completion queue entry  (refer to Figure 100), if any, for the Format NVM command
associated with this event. If no completion queue entry was reported, then this field shall be
cleared to 0h.
0 Phase Tag (P): This field may indicate the Phase Tag posted for the command.

11:10 Reserved
5.1.12.1.14.2.9 Sanitize Start Event (Event Type 09h)
A Sanitize Start event shall be recorded in the Persistent Event Log at the start of a sanitize operation (i.e.,
the sanitization target transitions to the Restricted Processing state or the Unrestricted Processing state,
as described in section 8.1.24.3).
The Sanitize Start event shall set the Persistent Event Log Event Format Header:
• Event Type field to 09h; and
• Event Type Revision field to 01h.
Figure 243: Sanitize Start Event Data Format (Event Type 09h)
Bytes Description
03:00 Sanitize Capabilities ( SANICAP): Contains the contents of the SANICAP field from the Identify
Controller data structure.
07:04 Sanitize CDW10 (SCDW10): Contains the value from command Dword 10 of the Sanitize command
(refer to Figure 372).
11:08 Sanitize CDW11 (SCDW11): Contains the value from command Dword 11 of the Sanitize command
(refer to Figure 373).
5.1.12.1.14.2.10 Sanitize Completion Event (Event Type 0Ah)
A Sanitize Completion event shall be recorded in the Persistent Event Log at the completion of a sanitize
operation (i.e., the sanitization target transitions to the Idle state, the Restricted Failure state, or the
Unrestricted Failure state, as described in section 8.1.24.3).
The Sanitize Completion event shall set the Persistent Event Log Event Format Header:
• Event Type field to 0Ah; and
• Event Type Revision field to 01h.
Figure 244: Sanitize Completion Event Data Format (Event Type 0Ah)
Bytes Description
1:0 Sanitize Progress (SPROG): Contains the sanitize progress at the time of this event using the format
specified for the SPROG field in the Sanitize Status log page (refer to section 5.1.12.1.33).
3:2
Sanitize Status (SSTAT): Contains the sanitize status for the time of this event using the format specified
for the SSTAT field in the Sanitize Status log page. (e.g., the Global Data Erase bit indicates the status
at the time of this event).
5:4
Completion Information (CINFO): Contains a vendor specific value that may provide more information
about the completion of the sanitize operation (e.g., if the sanitize operation did not complete successfully,
then this field may contain a vendor specific code that indicates a vendor specific reason).
7:6 Reserved
