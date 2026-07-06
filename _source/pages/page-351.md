---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 351
headings: []
---

# Source page 351

NVM Express® Base Specification, Revision 2.1

329
Figure 313: Identify – Power State Descriptor Data Structure
Bits Description
211:208
Emergency Power Fail Recovery Time Scale (EPFRTS):  This field indicates the scale for the
Emergency Power Fail Recovery Time field as defined in Figure 314.
If the EPFRT field is cleared to 0h, then this field:
• shall be cleared to 0h; and
• is ignored by the host.
207:200
Emergency Power Fail Vault Time (EPFVT): This field, with the EPFVTS field, indicates the worst-case
time required for the controller to complete Emergency Power Fail Processing (refer to section 8.2.5.3)
in the absence of failures. The time is equal to the value in this field multiplied by the scale indicated by
the EPFVTS field.
If Power Loss Signaling with Emergency Power Fail is not supported (i.e., the PLSEPF bit is cleared to
‘0’ in the Power Loss Signaling Information field of the Identify Controller data structure (refer to Figure
312)), then this field shall be cleared to 0h.
Value Definition
0 Not reported
1 to 99 Time value
100 to 255 Reserved

199:192
Forced Quiescence Vault Time (FQVT): This field, with the FQVTS field, indicates the worst-case time
required for the controller to complete Forced Quiescence Processing (refer to section 8.2.5.2). The time
is equal to the value in this field multiplied by the scale indicated in the FQVTS field.
If Power Loss Signaling with Forced Quiescence is not supported (i.e., the PLSFQ bit is cleared to ‘0’ in
the Power Loss Signaling Information field of the Identify Controller data structure (refer to Figure 312)),
then this field shall be cleared to 0h.
Value Definition
0 Not reported
1 to 99 Time value
100 to 255 Reserved

191:184
Emergency Power Fail Recovery Time (EPFRT): This field, with the EPFRTS field, indicates the worst-
case time required for the controller to complete the first initialization (as described in section 8.2.5.3)
after an Emergency Power Fail process which completed before power loss. The time is equal to the
value in this field multiplied by the scale indicated in the EPFRTS field.
If Power Loss Signaling with Emergency Power Fail is not supported (i.e., the PLSEPF bit is cleared to
‘0’ in the Power Loss Signaling Information field of the Identify Controller data structure (refer to Figure
312)), then this field shall be cleared to 0h.
Value Definition
0 Not reported
1 to 99 Time value
100 to 255 Reserved

183:182
Active Power Scale (APS): This field indicates the scale for the Active Power field. If an Active Power
Workload is reported for a power state, then the Active Power Scale shall also be reported for that power
state.
Value Definition
00b Not reported for this power state
01b 0.0001 W
10b 0.01 W
11b Reserved

181:179 Reserved
178:176
Active Power Workload (APW): This field indicates the workload used to calculate maximum power for
this power state. Refer to section 8.1.17.3 for more details on each of the defined workloads.  This field
shall not be “No Workload” unless ACTP is 0h.
