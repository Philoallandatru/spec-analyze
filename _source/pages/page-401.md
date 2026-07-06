---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 401
headings: []
---

# Source page 401

NVM Express® Base Specification, Revision 2.1

379
The Autonomous Power State Transition uses Command Dword 11 and specifies the attribute information
in the data st ructure indicated in  Figure 392 and the Autonomous Power State Transition data structure
consisting of 32 of the entries defined in Figure 393.
If a Get Features command is issued for this Feature, the attributes specified in Figure 392 are returned in
Dword 0 of the completion queue entry and the Autonomous Power State Transition data structure, whose
entry structure is defined in Figure 393, is returned in the data buffer for that command.
Figure 392: Autonomous Power State Transition – Command Dword 11
Bits Description
31:01 Reserved
00
Autonomous Power State Transition Enable (APSTE): This bit specifies whether autonomous power
state transition is enabled. If this bit is set to ‘1’, then autonomous power state transitions are enabled. If
this bit is cleared to ‘0’, then autonomous power state transitions are disabled.  This bit is cleared to ‘0’
by default.
Each entry in the Autonomous Power State Transit ion data structure is defined in Figure 393. Each entry
is 64 bits in size. There is an entry for each of the allowable 32 power states. For power states that are not
supported, the unused Autonomous Power State Transition data structure entries shall be cleared to all
zeroes. The entries begin with power state 0 and then increase sequentially (i.e., power state 0 is described
in bytes 7:0, power state 1 is described in bytes 15:8, etc.). The data structure is 256 bytes in size and shall
be physically contiguous.
Figure 393: Autonomous Power State Transition – Data Structure Entry
Bits Description
63:32 Reserved
31:08
Idle Time Prior to Transition (ITPT): This field specifies the amount of idle time that occurs in this power
state prior to transitioning to the Idle Transition Power State.  The time is specified in milliseconds.  A
value of 0h disables the autonomous power state transition feature for this power state.
07:03
Idle Transition Power State (ITPS): This field specifies the power state to which the controller
autonomously transitions, after there is a continuous period of idle time in the current power state that
exceeds the time specified in the Idle Time Prior to Transition  (ITPT) field. If the ITPT field is set to a
non-zero value, then the state specified in this field shall be a non -operational state as described in
Figure 313. This field should not specify a power state with higher reported idle power than the current
power state. If the ITPT field is cleared to 0h, then this field should be cleared to 0h.
02:00 Reserved
The Autonomous Power State Transition feature may interact with the Non-Operational Power State Config
feature (refer to section 5.1.25.1.10). Figure 394 shows these interactions.
Figure 394: Interactions between APSTE and NOPPME
APSTE1 NOPPME2 Non-operational power state entry Background operations during
non-operational power states
1 1 Entered by host request3 or by ITPT idle timer4 Allowed
0 1 Entered by host request3 Allowed
1 0 Entered by host request3 or by ITPT idle timer4 Not allowed
0 0 Entered by host request3 Not allowed
Notes:
1. Defined in Figure 392.
2. Defined in Figure 399.
3. Refer to section 5.1.25.1.2.
4. Refer to Figure 393.
