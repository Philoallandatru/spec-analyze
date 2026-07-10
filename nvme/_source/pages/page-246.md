---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 246
headings: ["5.1.12.1.10 Endurance Group Information (Log Page Identifier 09h)"]
---

# Source page 246

NVM Express® Base Specification, Revision 2.1

224
Figure 216: Telemetry Controller-Initiated Log Page
Bytes Description
382
Telemetry Controller-Initiated Data Available (TCDA): If this field is cleared to 0h, then
the log page consists of only the 512 -byte header and does not contain saved internal
controller state or saved internal NVM subsystem state available to be reported in response
to a Get Log Page command issued to the Admin Submission Queue.
If this field is set to 1h, then the log page contains saved internal controller state or saved
internal NVM subsystem state to be reported in response to a Get Log Page command
issued to the Admin Submission Queue. If this field is set to 1h, then it shall not be cleared
to 0h until a Get Log Page command with Retain Asynchronous Event bit cleared to ‘0’ for
the Telemetry Controller-Initiated log page completes successfully. This value is persistent
across power states and reset.
Regardless of the value of this field, the log page may contain saved internal controller state
or saved internal NVM subsystem state available to be reported in response to a Get Log
Page command issued out-of-band to a Management Endpoint (refer to the NVM Express
Management Interface Specification).
Other values are reserved.
383
Telemetry Controller -Initiated Data Generation Number  (TCDGN): Contains a value
that is incremented each time the internal controller state or the NVM subsystem state for
this log page is captured . If the value of this field is FFh, then the field shall be cleared to
0h when incremented (i.e., rolls over to 0h). This field is persistent across power cycles.
511:384
Reason Identifier (RID): Contains a vendor specific identifier that describes the operating
conditions of the controller at the time of capture. The Controller-Initiated Reason Identifier
field should provide an identification of unique operating conditions of the controller.
1023:512 Telemetry Controller-Initiated Data Block 1 (TCDB1): Contains Telemetry Data Block 1
for the Telemetry Controller -Initiated Log captured at a vendor specific time.
1535:1024 Telemetry Controller-Initiated Data Block 2 (TCDB2): Contains Telemetry Data Block 2
for the Telemetry Controller -Initiated Log captured at a vendor specific time.
… …
(n*512)+511:(n*512) Telemetry Controller-Initiated Data Block n (TCDBN): Contains Telemetry Data Block n
for the Telemetry Controller-Initiated log captured at a vendor specific time.
5.1.12.1.10 Endurance Group Information (Log Page Identifier 09h)
This log page is used to provide endurance information based on the Endurance Group (refer to section
3.2.3). An Endurance Group contains capacity that may be allocated to zero or more NVM Sets. Capacity
that has not been allocated to an NVM Set is unallocated Endurance Group capacity. The information
provided is over the life of the Endurance Group. The Endurance Group Identifier is specified in the Log
Specific Identifier field in Command Dword 11 of the Get Log Page command as defined in Figure 217. The
log page is 512 bytes in size.
Figure 217: Endurance Group Identifier - Log Specific Identifier
Bits Description
15:00 Endurance Group Identifier (ENDGID): This field specifies the identifier for the Endurance
Group (refer to section 3.2.3) used for this log page.
