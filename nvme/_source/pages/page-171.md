---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 171
headings: ["4.2.4 Phase Tag", "4.2.4.1 Phase Tag Example"]
---

# Source page 171

NVM Express® Base Specification, Revision 2.1

149
Figure 107: Status Code – Path Related Status Values
Value Description
04h to 5Fh Reserved
Controller detected Pathing errors
60h Controller Pathing Error: A pathing error was detected by the controller.
61h to 6Fh Reserved
Host detected Pathing errors
70h Host Pathing Error: A pathing error was detected by the host.
71h Command Aborted By Host: The command was aborted as a result of host action (e.g., the host
disconnected the fabric connection).
72h to 7Fh Reserved
Other Pathing errors
80h to BFh I/O Command Set Specific
C0h to FFh Vendor Specific
 Phase Tag
The Phase Tag bit indicates whether a completion queue entry is new. The Phase Tag bit for each
completion queue entry in:
• the Admin Completion Queue shall be initialized to ‘0’ by the host prior to setting CC.EN (refer to
Figure 41) to ‘1’; and
• an I/O Completion Queue shall be initialized to ‘0’ by the host prior to submitting the Create I/O
Completion Queue command for that queue.
When the controller posts a new completion queue entry to the Completion Queue, the controller shall
invert the Phase Tag bit in that completion queue entry (i.e., the inverting of the Phase Tag bit enables the
host to detect the new completion queue entry).
When a completion queue entry is posted to a completion queue slot in:
• the Admin Queue for the first time after CC.EN is set to ‘1’, the Phase Tag bit for that completion
queue entry is set to ‘1’; and
• an I/O Completion Queue for the first time after the Create I/O Completion Queue command
completed for that queue, the Phase Tag bit for that completion queue entry is set to ‘1’.
This continues for each completion queue entry that is posted until the controller posts a completion queue
entry to the highest numbered completion queue slot and wraps to completion queue slot number 0 as
described in section 3.3.1.2. When that queue wrap condition occurs, the Phase Tag bit is then cleared to
‘0’ in each completion queue entry that is posted. This continues until another queue wrap condition occurs.
Each time a queue wrap condition occurs, the value of the Phase Tag bit is inverted (i.e., changes from ‘1’
to ‘0’ or changes from ‘0’ to ‘1’).
 Phase Tag Example
Figure 108 shows an example of how the Phase Tag bit changes over time as a memory-based controller
completes commands and the host processes those completions. This example shows a Completion
Queue consisting of 6 entries.
