---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 264
headings: []
---

# Source page 264

NVM Express® Base Specification, Revision 2.1

242
Figure 236: NVM Subsystem Hardware Error Event Codes
Value Definition
06h
Critical Warning Condition: Indicates the NVM subsystem has detected a condition that causes a
bit in the Critical Warning field of the SMART / Health Information log (refer to section 5.1.12.1.3) to
be set to ‘1’.
Bits in this field represent the associated state at the time of this event.
The Additional Hardware Error Information field shall be set at the time of the event using the same
format as is specified for the Critical Warning field of the SMART / Health Information log.
07h
Endurance Group Critical Warning Condition: Indicates that the NVM subsystem has detected a
condition that causes a bit in the Critical Warning field of an Endurance Group Information log page
(refer to section 5.1.12.1.10) to be set to ‘1’.
Bits in this field represent the state at the time this event is added to the Persistent Event log page.
The Additional Hardware Error Information field shall be four bytes long  and contain the following
information:
Bytes Description
0
Endurance Group Critical Warning (EGCW): This field shall be set at the time this
event is added to the Persistent Event log page using the same format as is specified
for the Critical Warning field of the Endurance Group Information log page.
1 Reserved
3:2 Endurance Group Identifier (EGID): This field shall be set to the Endurance Group
Identifier for the associated Endurance Group.

08h
Unexpected Power Loss: Indicates that the controller incremented the Unexpected Power Losses
field value in the SMART / Health Information Log.
Note that this field was previously named Unsafe Shutdown.
Refer to Figure 237 for the format of the Additional Hardware Error Information field.
09h
Controller Fatal Status: Indicates that the Controller Fatal Status (CSTS.CFS) bit has been set to
‘1’.
This NVM subsystem hardware error event does not contain additional hardware error information.
0Ah
Media and Data Integrity Status:  Indicates that a completion queue entry contained a Media and
Data Integrity status code (refer to Figure 106) other than 86h (i.e., Access Denied) or 87h (i.e.,
Deallocated or Unwritten logical Block).
The Additional Hardware Error Information field shall be set to the contents of the completion queue
entry.
0Bh
Controller Ready Timeout Exceeded: Indicates that:
a) the controller was not ready to process at least one command without error as described in
section 3.5.4.1 within the amount of time indicated by:
• the Controller Ready With Media Timeout (CRTO.CRWMT)  field in Controller
Ready With Media mode (CC.CRIME is cleared to ‘0’); or
• the Controller Ready Independent of Media Timeout (CRTO.CRIMT)  field in
Controller Ready Independent of Media mode (CC.CRIME is set to ‘1’),
 since the controller was enabled; or
b) at least one namespace attached to the controller or media required to process at least one
Admin command was not ready within the amount of time indicated by the Controller Ready
With Media Timeout (CRTO.CRWMT) field since the controller was enabled by transitioning
CC.EN from ‘0’ to ‘1’.
Refer to Figure 239 for the format of the Additional Hardware Error Information field.
0Ch to FFh Reserved
