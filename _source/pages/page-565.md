---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 565
headings: ["8.1.18.3 Configuring and Managing Events", "8.1.19 Reachability Reporting architecture", "8.1.19.1 Reachability Reporting overview"]
---

# Source page 565

NVM Express® Base Specification, Revision 2.1

543
The DTWIN Time Maximum and NDWIN Time Minimum High may provide a ratio of the amount of
maintenance that needs to be performed based on the time that the NVM Set remains in the DTWIN,
assuming no threshold is exceeded. Any scaling of the time in the Non-Deterministic Window based on the
read, write, and time behavior in the previous Deterministic Window is implementation dependent.
The DTWIN Time Estimate may be used by the host when a Deterministic Excursion has occurred. This
estimate allows the host to re -synchronize an NVM Set with other NVM Sets operating in Predictable
Latency Mode, if applicable.
 Configuring and Managing Events
The host may configure events to be triggered when thresholds do not exceed certain levels or when
autonomous transitions occur using the Predictable Latency Mode Feature. The host submits a Set Feature
command for the particular NVM Set and configures the specific event(s) and threshold(s) values that shall
trigger a Predictable Latency Event Aggregate Log Change Notice event for that particular NVM Set to the
host. Refer to Figure 404.
The host determines the NVM Sets that have outstanding events by reading the Predictable Latency Event
Aggregate log page (refer to section 5.1.12.1.12). An entry is returned for each NVM Set that has an event
outstanding. The host may use the NVM Set Identifier Maximum value reported in the Identify Controller
data structure in order to determine the maximum size of this log page.
To determine the specific event(s) that have occurred for a reported NVM Sets, the host reads the
Predictable Latency Per NVM Set log page (refer to section 5.1.12.1.11) for that NVM Set. The Event Type
field indicates the event(s) that have occurred (e.g., an autonomous transition to the NDWIN). An event(s)
for a particular NVM Set is cleared if the controller successfully processes a read for the Predictable Latency
Per NVM Set log page for the affected NVM Set where the Get Log Page command has the Retain
Asynchronous Event parameter cleared to ‘0’. If the Event Type field in the Predictable Latency Per NVM
Set log page is cleared to 0h, then events for that particular  NVM Set are not reported in the Predictable
Latency Event Aggregate log page.
 Reachability Reporting architecture
 Reachability Reporting overview
For some operations that specify multiple namespaces it is necessary to indicate what namespaces are
able to be used in those operations (e.g., Copy command that specifies multiple namespaces, Execute
command in the Computational Programs command set). Thi s ability is called reachability. Reachability is
defined at the controller level (i.e., only namespaces attached to the same controller are reachable).
A Reachability Group becomes available to a controller if:
a) a namespace becomes attached to that controller and that namespace is associated with a
Reachability Group that did not currently have any namespaces attached to that controller; or
b) a configuration change in an NVM subsystem changes the membership in a Reachability Group
(e.g., a new Reachability Group is present in the Reachability Groups log page (refer to section
5.1.12.1.25)).
A Reachability Group becomes unavailable to a controller if a namespace is detached that is the only
namespace attached to that controller for that Reachability Group (i.e., a Reachability Group that was
present in the Reachability Groups log page is no longer present).
A Reachability Association change occurs if a Reachability Group that is associated with that Reachability
Association becomes available or unavailable to that controller. (e.g., a new Reachability Association is
present in the Reachability Associations log page (refer to section 5.1.12.1.26) or a Reachability Association
that was present in the Reachability Associations log page is no longer present).
If an NVM subsystem supports Reachability Reporting, then all controllers in that NVM subsystem shall:
• set the RRSUP bit to ‘1’ in the Controller Reachability Capabilities (CRCAP) field in the Identify
Controller data structure (refer to Figure 312) to indicate support for Reachability Reporting;
• support Reachability Groups Change Notices (refer to section 5.1.25.1.5);
