---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 232
headings: []
---

# Source page 232

NVM Express® Base Specification, Revision 2.1

210
Figure 206: SMART / Health Information Log Page
Bytes Description
06
Endurance Group Critical Warning Summary (EGCWS): This field indicates critical warnings for the
state of Endurance Groups. Each bit corresponds to a critical warning type , multiple bits may be set to
‘1’. If a bit is cleared to ‘0’, then that critical warning does not apply to any Endurance Group. Critical
warnings may result in an asynchronous event notification to the host. Bits in this field represent the
current associated state and are not persistent.
If a bit is set to ‘1’ in one or more Endurance Groups, then the corresponding bit shall be set to ‘1’ in this
field.
Bits Description
7:4 Reserved
3
Endurance Group Read -Only (EGRO): If this bit is set to ‘1’, then the namespaces in
one or more Endurance Groups have been placed in read only mode not as a result of a
change in the write protection state of a namespace (refer to section 8.1.16.1).
2
Endurance Group Degraded Reliability (EGDR): If this bit is set to ‘1’, then the reliability
of one or more Endurance Groups has been degraded due to significant media related
errors or any internal error that degrades NVM subsystem reliability.
1 Reserved
0
Endurance Group Available Spare Capacity Below Threshold (EGASCBT): If this bit
is set to ‘1’, then the available spare capacity of one or more Endurance Groups has fallen
below the threshold.

31:07 Reserved
47:32
Data Units Read  (DUR): Contains the number of 512 byte data units the host has read from the
controller as part of processing a SMART Data Units Read  Command; this value does not include
metadata. This value is reported in thousands (i.e., a value of 1 corresponds to 1,000 units of 512 bytes
read) and is rounded up  (e.g., one indicates the that number of 512 byte data units read is from 1 to
1,000, three indicates that the number of 512 byte data units read is from 2,001 to 3,000).
Refer to the specific I/O Command Set specification for the list of SMART Data Units Read Commands
that affect this field.
A value of 0h in this field indicates that the number of SMART Data Units Read is not reported.
63:48
Data Units Written  (DUW): Contains the number of 512 byte data units the host has written to the
controller as part of processing a User Data Out Command; this value does not include metadata. This
value is reported in thousands (i.e., a value of 1 corresponds to 1,000 units of 512 bytes written) and is
rounded up (e.g., one indicates that the number of 512 byte data units written is from 1 to 1,000, three
indicates that the number of 512 byte data units written is from 2,001 to 3,000).
Refer to the specific I/O Command Set specification for the list of User Data Out Commands that affect
this field.
A value of 0h in this field indicates that the number of Data Units Written is not reported.
79:64
Host Read Commands (HRC): Contains the number of SMART Host Read Commands completed by
the controller.
Refer to the specific I/O Command Set specification for the list of SMART Host Read Commands that
affect this field.
95:80
Host Write Commands (HWC): Contains the number of User Data Out Commands completed by the
controller.
Refer to the specific I/O Command Set specification for the list of User Data Out Commands that affect
this field.
111:96
Controller Busy Time (CBT): Contains the amount of time the controller is busy with I/O commands.
The controller is busy when there is a command outstanding to an I/O Queue (specifically, a command
was issued via an I/O Submission Queue Tail doorbell write and the corresponding completion queue
entry has not been posted yet to the associated I/O Completion Queue).  This value is reported in
minutes.
127:112 Power Cycles (PWRC): Contains the number of power cycles.
143:128 Power On Hours (POH): Contains the number of power -on hours. This may not include time that the
controller was powered and in a non-operational power state.
