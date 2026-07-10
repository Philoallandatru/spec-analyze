---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 196
headings: []
---

# Source page 196

NVM Express® Base Specification, Revision 2.1

174
Request commands are outstanding when the controller is reset, then each of those commands is aborted
and shall not return a CQE.
All command specific fields are reserved.
A host may submit multiple Asynchronous Event Request commands  to reduce event reporting latency.
The total number of simultaneously outstanding Asynchronous Event Request commands is limited by the
value indicated in the Asynchronous Event Request Limit  field in the Identify Controller data structure  in
Figure 312.
Asynchronous events are grouped into event types. The event type is indicated in the Asynchronous Event
Type field in Dword 0 of the completion queue entry for the Asynchronous Event Request command. When
the controller posts a completion queue entry for an outstanding Asynchronous Event Request command
and thus reports an asynchronous event, then, except for immediate events, subsequent events of that
event type are automatically masked by the controller until the host clears that event . Unless otherwise
stated (e.g., for immediate events), an event is cleared by reading the log page associated with that event
(refer to section 5.1.12). If that log page is not accessible because media is not ready (i.e., the controller
aborts the Get Log Page command with  a status code of Admin Command Media Not Ready), then the
controller shall not post a completion queue entry for that asynchronous event until the controller is able to
successfully return the log page that is required to be read to clear the asynchronous  event.
The following event types are defined:
a) Error event:  Indicates a general error that is not associated with a specific command  (refer to
Figure 149). The controller shall set the Log Page Identifier field to the identifier of the Error
Information log page (i.e., 01h). To clear this event,  a host reads that log page (i.e., the Error
Information log page (refer to section 5.1.12.1.2)) using the Get Log Page command with the Retain
Asynchronous Event bit cleared to ‘0’;
b) SMART / Health Status event:  Indicates a SMART or health status event  (refer to Figure 150).
The controller shall set the Log Page Identifier field to the identifier of the SMART/Health
Information log page (i.e., 02h). To clear this event, a host reads the SMART / Health Information
log (refer to section 5.1.12.1.3) using the Get Log Page command with the Retain Asynchronous
Event bit cleared to ‘0’. The SMART / Health conditions that trigger asynchronous events may be
configured in the Asynchronous Event Configuration feature using the Set Features  command
(refer to section 5.1.25.1.5);
c) Notice event: Indicates a general event (refer to Figure 151). The controller shall set the Log Page
Identifier field to the log page identifier of the appropriate log page as described in Figure 151. To
clear this event, a host reads that log page (i.e., the appropriate log page as described in  Figure
151). The conditions that trigger asynchronous events may be configured in the Asynchronous
Event Configuration feature using the Set Features command (refer to section 5.1.25.1.5);
d) I/O Command Specific Status events: Events that are specific to an I/O command (refer to Figure
152);
e) Immediate events:  Events that are only reported when an outstanding Asynchronous Event
Request command exists at the time the event occurs (refer to Figure 153). If the event occurs and
there is no outstanding Asynchronous Event Request command, then the event shall not be
reported;
f) One-Shot events: Events that are only reported once and the event is cleared by the controller
when that event is reported in the completion of an Asynchronous Event Request command. Refer
to the description of each event to determine how to clear any pending events (e.g.,  refer to the
Controller Data Queue feature in section 5.1.25.1.23); and
g) Vendor Specific event:  Indicates a vendor specific event.  To clear this event, a host reads the
indicated vendor specific log page using the Get Log Page command with the Retain Asynchronous
Event bit cleared to ‘0’.
The Sanitize Operation Completed With Unexpected Deallocation asynchronous event shall be supported
if the controller supports the Sanitize Config feature (refer to section 5.1.25.1.15).
Asynchronous events are reported due to a new entry being added to a log page (e.g., Error Information
log page), a status change (e.g., status in the SMART / Health log  page), or occurrence of a defined
