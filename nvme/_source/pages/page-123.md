---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 123
headings: ["3.4.4.3 Vendor Specific Arbitration", "3.4.5 Outstanding Commands"]
---

# Source page 123

NVM Express® Base Specification, Revision 2.1

101
Figure 81: Weighted Round Robin with Urgent Priority Class Arbitration

In Figure 81, the Priority decision point selects the highest priority candidate command selected next to
start processing.
 Vendor Specific Arbitration
A vendor may choose to implement a vendor specific arbitration mechanism. The mechanism(s) are outside
the scope of this specification.
 Outstanding Commands
A command is outstanding if:
• the host has submitted that command to the controller;
• the host has not received a completion for that command; and
• as described in this section:
o the host has not performed an action that causes that command to no longer be outstanding;
and
o the host has not otherwise determined that that command is no longer outstanding .
A submitted command is no longer outstanding after the host:
• receives a completion for that command;
ASQ
SQ
SQ
SQ
RR
SQ
SQ
SQ
RR
SQ
SQ
SQ
RR
WRR
Priority
Weight(High)
Admin
High
Priority
Medium
Priority
Low
Priority
Weight(Medium)
Weight(Low)
Strict
Priority 2
SQ
SQ
RRUrgent
Strict
Priority 3
Strict
Priority 1
