---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 79
headings: []
---

# Source page 79

NVM Express® Base Specification, Revision 2.1

57
prior to enabling the controller by setting CC.EN to ‘1’. Attempting to create an I/O queue before initializing
the I/O Completion Queue Entry Size (CC.IOCQES) and the I/O Submission Queue Entry Size
(CC.IOSQES) shall cause a controller to abort a Create I/O Completion Queue command or a Create I/O
Submission Queue command with a status code of Invalid Queue Size.
Figure 41: Offset 14h: CC – Controller Configuration
Bits Type Reset Description
31:25 RO 0h Reserved
24 RW/RO 0b
Controller Ready Independent of Media Enable (CRIME): This bit controls
the controller ready mode. The controller ready mode is determined by the state
of this bit at the time the controller is enabled by transitioning the CC.EN bit from
‘0’ to ‘1’.
If the CAP.CRMS field is set to 11b, then this bit is RW. If the CAP.CRMS field
is not set to 11b, then this bit is RO and shall be cleared to ‘0’. Refer to sections
3.5.3 and 3.5.4 for more detail.
Changing the value of this field may cause a change in the time reported in the
CAP.TO field. Refer to the definition of CAP.TO for more details.
Value Definition
0b
Controller Ready With Media Mode: Enabling the controller (i.e.,
CC.EN transitions from ‘0’ to ‘1’) when this bit is cleared to ‘0’
enables Controller Ready With Media mode.
1b
Controller Ready Independent Of Media Mode: Enabling the
controller when this bit is set to ‘1’ enables Controller Ready
Independent of Media mode.

23:20 RW/RO 0h
I/O Completion Queue Entry Size (IOCQES):  This field defines the I/O
completion queue entry size that is used for the selected I/O Command Set(s).
The required and maximum values for this field are specified in the CQES field
in the Identify Controller data structure in Figure 312 for each I/O Command
Set. The value is in bytes and is specified as a power of two (2^n).
If any I/O Completion Queues exist, then write operations that change the value
in this field produce undefined results.
If the controller does not support I/O queues, then this field shall be read -only
with a value of 0h.
For Discovery controllers, this field is reserved.
19:16 RW/RO 0h
I/O Submission Queue Entry Size (IOSQES):  This field defines the I/O
submission queue entry size that is used for the selected I/O Command Set(s).
The required and maximum values for this field are specified in the SQES field
in the Identify Controller data structure in Figure 312 for each I/O Command
Set. The value is in bytes and is specified as a power of two (2^n).
If any I/O Submission Queues exist, then write operations that change the value
in this field produce undefined results.
If the controller does not support I/O queues, then this field shall be read -only
with a value of 0h.
For Discovery controllers, this field is reserved.
