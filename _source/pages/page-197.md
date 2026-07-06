---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 197
headings: ["5.1.2.1 Command Completion"]
---

# Source page 197

NVM Express® Base Specification, Revision 2.1

175
condition (e.g., Firmware Activation Starting, NVM Subsystem Normal Shutdown). A status change may be
permanent (e.g., the media has become read only) or transient (e.g., the temperature reached or exceeded
a threshold for a period of time). A host should modify the event threshold or mask the event for transient
and permanent status changes before clearing that event  to avoid repeated reporting of asynchronous
events caused by clearing an asynchronous event in the absence of change to the status that cause d that
event to be reported.
If an event occurs for which reporting is enabled and there are no Asynchronous Event Request commands
outstanding, the controller should retain the event information for that Asynchronous Event Type (i.e., a
pending event) and use that information as a response to the next Asynchronous Event Request command
that is received. If a Get Log Page command clears the event prior to receiving the Asynchronous Event
Request command or if a Controller Level Reset occurs, then a notification is not sent. If multiple events of
the same type occur that have identical responses to the Asynchronous Event Request command, then
those events may be reported as a single response to an Asynchronous Event Request command. If
multiple events occur that are of different types  or have different responses to the Asynchronous Event
Request command, then the controller should retain a queue of those events for reporting in responses to
subsequent Asynchronous Event Request commands.
 Command Completion
A completion queue entry is posted to the Admin Completion Queue  if there is an asynchronous event to
report to the host.  Command specific status values associated with Asynchronous Event Request  are
defined in Figure 146.
Figure 146: Status Code – Command Specific Status Values
Value Definition
05h Asynchronous Event Request Limit Exceeded:  The number of concurrently outstanding
Asynchronous Event Request commands has been exceeded.
Dword 0 and Dword 1 of the completion queue entry contains information about the asynchronous event.
The definition of Dword 0 of the completion queue entry is in Figure 147. The definition of Dword 1 of the
completion queue entry is in Figure 148.
Figure 147: Asynchronous Event Request – Completion Queue Entry Dword 0
Bits Description
31:24 Reserved
23:16
Log Page Identifier (LID): Indicates the log page associated with the asynchronous event. This log page
needs to be read by the host to clear the event.
For asynchronous events not associated with a log page (refer to section 5.1.2), this field should be
ignored by the host.
15:08 Asynchronous Event Information  (AEI): This field indicates additional information about the
asynchronous event for the event indicated in the Asynchronous Event Type field.
07:03 Reserved
02:00
Asynchronous Event Type (AET): Indicates the event type of the asynchronous event.
Value Definition Reference
000b Error status Figure 149
001b SMART / Health status Figure 150
010b Notice Figure 151
011b Immediate Figure 153
100b One-Shot Figure 154
101b Reserved
110b I/O Command specific status Figure 152
111b Vendor specific
