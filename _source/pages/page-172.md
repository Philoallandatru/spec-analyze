---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 172
headings: []
---

# Source page 172

NVM Express® Base Specification, Revision 2.1

150
 Figure 108: Phase Tag bit Transition Example
T1 Condition Completion Queue Entry/Slot number
0 1 2 3 4 5
0
Admin Queue: Host initializes
Completion Queue and sets CC.EN
to ‘1’
I/O Queue: Host initializes
Completion Queue and submits
Create I/O Completion Queue
command
P(0) (E)
HEAD->
TAIL->
P(0)
(E)
P(0)
(E)
P(0)
(E)
P(0)
(E)
P(0)
(E)
1
Controller has completed 1
command and the host has
consumed 0 completions
P(1)
HEAD->
P(0) (E)
TAIL->
P(0)
(E)
P(0)
(E)
P(0)
(E)
P(0)
(E)
2
Controller has completed 6
commands and the host has
consumed 2 completions
P(1) (E)
TAIL->
P(1)
(E)
P(1)
HEAD-> P(1) P(1) P(1)
3
Controller has completed 7
commands and the host has
consumed 2 completions
P(0) P(1) (E)
TAIL->
P(1)
HEAD-> P(1) P(1) P(1)
4
Controller has completed 7
commands and the host has
consumed 4 completions
P(0) P(1) (E)
TAIL->
P(1)
(E)
P(1)
(E)
P(1)
HEAD-> P(1)
5
Controller has completed 11
commands and the host has
consumed 8 completions
P(0)
(E)
P(0)
(E)
P(0)
HEAD-> P(0) P(0) P(1) (E)
TAIL->
6
Controller has completed 11
commands and the host has
consumed 11 completions
P(0)
(E)
P(0)
(E)
P(0)
(E)
P(0)
(E)
P(0)
(E)
P(1) (E)
HEAD->
TAIL->
Key:
P(0) = Phase Tag bit for this completion queue entry is cleared to the value ‘0’.
P(1) = Phase Tag bit for this completion queue entry is set to the value ‘1’.
(E) = The Entry/Slot is empty.
HEAD-> = Completion Queue Head Pointer for this completion queue is set to indicate this slot.
TAIL-> = Completion Queue Tail Pointer for this completion queue is used within the controller to indicate this
slot.
Note:
1. T = Time sequence.
At time 0, the host initializes the Completion queue (i.e., clearing the Phase Tag bit to ‘0’ in each completion
queue entry in the completion queue). For the Admin Completion Queue, the host then sets CC.EN to ‘1’
to enable the controller. For an I/O Comp letion Queue, the host then sends the Create I/O Completion
Queue command. The queue, at this time, is in the Empty condition (refer to section 3.3.1.4).
At time 1, the controller has completed a command, but the host has not consumed that completion queue
entry. As a result of the command completion, the Phase Tag bit in completion queue entry 0 has been
inverted to ‘1’. Since no completion queue entries have been consumed, the Completion Queue Head
pointer still indicates completion queue entry 0. The controller has updated the internal Completion Queue
Tail Pointer to indicate that completion queue slot 1 is the next completion q ueue slot into which the
controller posts a completion queue entry.
At time 2, the controller has completed 5 additional commands (i.e., 6 commands have been completed)
and the host has consumed 2 of the completion queue entries. As a result of the 5 additional commands
having been completed, the Phase Tag bit has been inv erted to ‘1’ in completion queue entry 1 through
completion queue entry 5. As a result of 2 completion queue entries having been consumed, the host has
updated the Completion Queue Head Pointer to indicate that completion queue entry 0 and completion
queue entry 1 have been consumed (i.e., completion queue entry 2 is the next completion queue entry for
the host to consume). The controller has updated the internal Completion Queue Tail Pointer to indicate
