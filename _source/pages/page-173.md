---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 173
headings: ["4.3 Data Pointer Layouts (PRPs and SGLs)", "4.3.1 Physical Region Page Entry and List"]
---

# Source page 173

NVM Express® Base Specification, Revision 2.1

151
that completion queue slot 0 is the next completion queue slot into which the controller posts a completion
queue entry.
At time 3, the controller has completed 1 additional command (i.e., 7 commands have been completed) and
no additional completion queue entries have been consumed by the host (i.e., 2 completion queue entries
have been consumed). As a result of the addition al command having been completed, the Phase Tag bit
has been inverted to ‘0’ in completion queue entry 0 (i.e., accounting for the queue wrap condition). The
controller has updated the internal Completion Queue Tail Pointer to indicate that completion queu e slot 1
is the next completion queue slot into which the controller posts a completion queue entry . The queue, at
this time, is in the Full condition (refer to section 3.3.1.5).
At time 4, the controller has completed no additional commands (i.e., 7 commands have been completed)
and the host has consumed 2 additional completion queue entries (i.e., 4 completion queue entries have
been consumed). As a result of 2 additional completion queue entries having been consumed, the host has
updated the Completion Queue Head Pointer to indicate that completion queue entry 2 and completion
queue entry 3 have now been consumed (i.e., completion queue entry 4 is the next completion queue entry
for the host to consume). The controller internal Completion Queue Tail Pointer has not changed.
At time 5, the controller has completed 11 commands and the host has consumed 8 of the completion
queue entries. As a result of the 4 additional commands having been completed, the Phase Tag bit has
been inverted to ‘0’ in completion queue entry 1 through completion queue entry 4. As a result of the 4
additional completion queue entries having been consumed, the host has updated the Completion Queue
Head Pointer to indicate that completion queue entry 5 through completion queue entry 1 (i.e., accounting
for the queue wrap condition) have now been consumed (i.e., completion queue entry 2 is the next
completion queue entry for the host to consume). The controller has updated the internal Completion Queue
Tail Pointer to indicate that completion queue slot 5 is  the next completion queue slot into which the
controller posts a completion queue entry.
At time 6, the controller has completed 11 commands and the host has consumed all 11 of the completion
queue entries. As a result of no new command completions, there are no changes to the Phase Tag bit
values. As a result of the 3 additional completion qu eue entries having been consumed, the host has
updated the Completion Queue Head Pointer to indicate that completion queue entry 2 through completion
queue entry 4 have now been consumed (i.e., completion queue slot 5 is the next completion queue slot
from which the host consumes a completion queue entry). The queue, at this time, is in the Empty condition
(refer to section 3.3.1.4).
4.3 Data Pointer Layouts (PRPs and SGLs)
This section describes the data structures used to describe the layout of data that can be understood by
the controller and the host.
 Physical Region Page Entry and List
A physical region page (PRP) entry is a pointer to a physical memory page. PRPs are used as a
scatter/gather mechanism for data transfers between the controller and memory. To enable efficient out of
order data transfers between the controller and the host, PRP entries are a fixed size.
The size of the physical memory page is configured by host software in CC.MPS. Figure 109 shows the
layout of a PRP entry that consists of a Page Base Address and an Offset. The size of the Offset field is
determined by the physical memory page size configured in CC.MPS.
Figure 109: PRP Entry Layout
63 n+1 n 0
Page Base Address Offset
The definition of a PRP entry is described in Figure 110.
