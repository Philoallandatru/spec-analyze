---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 253
headings: []
---

# Source page 253

NVM Express® Base Specification, Revision 2.1

231
A sanitize operation may alter this log page (e.g., remove or modify events to prevent derivation of user
data from log page information, refer to section 8.1.24). The events removed from this log page by a sanitize
operation are unspecified.
Persistent Event Log events specified in this section should be reported in an order such that more recent
events are generally reported earlier in the log data than older events. The method by which the NVM
subsystem determines the order in which events occurred is vendor specific.
The number of events supported is vendor specific. The supported maximum size for the Persistent Event
Log is indicated in the PELS field of the Identify Controller data structure (refer to Figure 312). The number
of events supported and the supported maximum size should be large enough that the number of events
or the size of the Persistent Event Log data does not reach the maximum supported size over the usable
life of the NVM subsystem.
The controller shall log all supported events at each event occurrence unless the controller determines that
the same event is occurring at a frequency that exceeds a vendor specific threshold for the frequency of
event creation. If the same event is occurring at a frequency that exceeds a vendor specific threshold then
the vendor may suppress further entries for the same event. A controller may indicate if events have been
suppressed in vendor specific event data.
It is vendor specific which events are deleted (e.g., important events may be retained and events that are
newer than an important event that was retained may be deleted) to make room for future events if:
a) the size of the Persistent Event Log data reaches the maximum supported size;
b) the number of events reaches the largest reportable number of events; or
c) an event category reaches the largest reportable number of events for that category (e.g.,
information regarding 1,000 occurrences of changes to the timestamp is stored in internal data
structures and extracted for reporting as Timestamp Change events i n the Persistent Event Log
and more than 1,000 Timestamp Change events have occurred).
Events that affect multiple controllers (e.g., an NVM Subsystem Reset) should be logged once by a
controller selected by the vendor and not logged by any other controllers.
The Action field in the Log Specific Parameter field (refer to Figure 225) specifies whether:
a) A persistent event log reporting context is created at the start of processing this Get Log Page
command and log page data, if any, is read from the log page data associated with that log reporting
context;
b) Log page data is read from the log page data associated with a preexisting log reporting context;
or
c) The persistent event log reporting context, if any, is released.
The persistent event log reporting context is vendor specific information that the controller creates for
determining what information is included in the persistent event log page data (e.g., the persistent event
log reporting context may be the persistent event log page data or may contain a set of pointers to the
events to report).
The controller should retain the persistent event log reporting context:
a) Until the controller processes:
a) a Get Log Page command requesting the Persistent Event log page with the Action field set to
02h (i.e., Release Context);
b) an NVM Subsystem Reset; or
c) a Controller Level Reset;
or
b) For a vendor specific time long enough to allow retrieval of the Persistent Event log page data.
Persistent Event Log events that occur while a persistent event log reporting context exists shall not be
reported in the existing reporting context but shall be logged.
