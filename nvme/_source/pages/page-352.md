---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 352
headings: []
---

# Source page 352

NVM Express® Base Specification, Revision 2.1

330
Figure 313: Identify – Power State Descriptor Data Structure
Bits Description
175:160
Active Power (ACTP): This field indicates the largest average power consumed by the NVM subsystem
over a 10 second period in this power state with the workload indicated in the Active Power Workload
field. The power in Watts is equal to the value in this field multiplied by th e scale indicated in the Active
Power Scale field. A value of 0h indicates Active Power is not reported.
159:152 Reserved
151:150
Idle Power Scale (IPS): This field indicates the scale for the Idle Power field.
Value Definition
00b Not reported for this power state
01b 0.0001 W
10b 0.01 W
11b Reserved

149:144 Reserved
143:128
Idle Power (IDLP):  This field indicates the typical power consumed by the NVM subsystem over 30
seconds in this power state when idle ( e.g., there are no pending commands, property accesses,
background processes, sanitize operation, nor device self-test operations). The measurement starts after
the NVM subsystem has been idle for 10 seconds.  The power in Watts is equal to the value in this field
multiplied by the scale indicated in the Idle Power Scale field.  A value of 0h indicates Idle Power is not
reported. Refer to section 8.1.17.
Note: This value may be used by hosts to manage power versus resume latency. Platform and form factor
specifications may have additional power measurement and reporting requirements that are outside the
scope of this specification.
127:125 Reserved
124:120
Relative Write Latency (RWL): This field indicates the relative write latency associated with this power
state. The value in this field shall be less than the number of supported power states (e.g., if the controller
supports 16 power states, then valid values are 0 through 15). A lower value means lower write latency.
119:117 Reserved
116:112
Relative Write Throughput (RWT): This field indicates the relative write throughput associated with this
power state. The value in this field shall be less than the number of supported power states (e.g., if the
controller supports 16 power states, then valid values are 0 through 15). A l ower value means higher
write throughput.
111:109 Reserved
108:104
Relative Read Latency (RRL): This field indicates the relative read latency associated with this power
state. The value in this field shall be less than the number of supported power states (e.g., if the controller
supports 16 power states, then valid values are 0 through 15). A lower value means lower read latency.
103:101 Reserved
100:96
Relative Read Throughput (RRT): This field indicates the relative read throughput associated with this
power state. The value in this field shall be less than the number of supported power states (e.g., if the
controller supports 16 power states, then valid values are 0 through 15). A lo wer value means higher
read throughput.
95:64 Exit Latency (EXLAT): This field indicates the maximum exit latency in microseconds associated with
exiting this power state. A value of 0h indicates Exit Latency is not reported.
63:32 Entry Latency (ENLAT): This field indicates the maximum entry latency in microseconds associated with
entering this power state. A value of 0h indicates Entry Latency is not reported.
31:26 Reserved
25
Non-Operational State (NOPS):  This bit indicates whether the controller processes I/O commands in
this power state.  If this bit is cleared to ‘0’, then the controller processes I/O commands in this power
state. If this bit is set to ‘1’, then the controller does not process I/O commands in this power state. Refer
to section 8.1.17.1.
24
Max Power Scale (MXPS): This bit indicates the scale for the Maximum Power field. If this bit is cleared
to ‘0’, then the scale of the Maximum Power field is in 0.01 Watts. If this bit is set to ‘1’, then the scale of
the Maximum Power field is in 0.0001 Watts.
23:16 Reserved
