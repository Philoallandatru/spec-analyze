---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 103
headings: ["3.2.4 Reclaim Groups, Reclaim Unit Handles, and Reclaim Units"]
---

# Source page 103

NVM Express® Base Specification, Revision 2.1

81
The host determines the Endurance Groups that have outstanding events by reading the Endurance Group
Event Aggregate log page. An entry is returned for each Endurance Group that has an event outstanding.
The host may use the Endurance Group Identifier Maximum value reported in the Identify Controller data
structure to determine the maximum size of this log page.
To determine the specific event(s) that have occurred for a reported Endurance Group, the host reads the
Endurance Group Information log page (refer to Figure 218) for that Endurance Group. The Critical Warning
field indicates the event(s) that have occurred (e.g., that all namespaces in the Endurance Group have
been placed in read-only mode). All events for an Endurance Group are cleared if the controller successfully
processes a read for the Endurance Group Information log page for that Endurance Group, where the Get
Log Page command has the Retain Asynchronous Event bit cleared to ‘0’. If the Critical Warning field in the
Endurance Group Information log page is cl eared to 0h, then events for that Endurance Group are not
reported in the Endurance Group Event Aggregate log page.
 Reclaim Groups, Reclaim Unit Handles, and Reclaim Units
If Flexible Data Placement is enabled in an Endurance Group (refer to section 5.1.25.1.20), then the logical
view of the non-volatile storage capacity in that Endurance Group is shown in Figure 70 and consists of:
• a set of one or more Reclaim Groups numbered from 0 to P -1 where P is the value of the Number
of Reclaim Groups field in the FDP Configuration Descriptor (refer to Figure 280). A Reclaim Group
consists of one or more Reclaim Units; and
• one or more Reclaim Unit Handles numbered from 0 to N -1 where N is the value of the Number of
Reclaim Unit Handles field in the FDP Configuration Descriptor (refer to Figure 280).
A Reclaim Unit Handle consists of a reference to a Reclaim Unit in each Reclaim Group where user data
from a write command is placed. A Reclaim Unit referenced by the Reclaim Unit Handle is only allowed to
be referenced by at most one Reclaim Unit Handle. However, a specific Reclaim Unit is referenced by the
same or different Reclaim Unit Handles as the Reclaim Unit is cycled from erased and back into use. When
a Reclaim Unit is written to capacity, the controller updates that Reclaim Unit Handle to reference a different
Reclaim Unit that is available for writing user data (e.g., non -volatile storage media that has been erased
which is required prior to writing for program in place memories) and has not been written with any user
data (i.e., an empty Reclaim Unit). Refer to section 8.1.10 for the details of how a host is able to issue a
write command and place the user data into a Reclaim Unit.
