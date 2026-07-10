---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 618
headings: ["8.2.5.1.1 PLS Not Ready State", "8.2.5.1.2 PLS Ready State"]
---

# Source page 618

NVM Express® Base Specification, Revision 2.1

596
Figure 661: PLS States
State Name PLA Variable Value
(if supported)
Port
Communication
Processed
PLS Not Ready Deasserted Yes
PLS Ready Deasserted Yes
FQ Processing Asserted-FQ Yes
FQ Complete Deasserted Yes
EPF Processing Port Disabled1 Asserted-EPF-Disabled No
EPF Complete Port Disabled Deasserted No
EPF Processing Port Enabled1 Asserted-EPF-Enabled Yes
EPF Complete Port Enabled Deasserted Yes
Notes:
1. The controller shall not implement both the EPF Processing Port Disabled state and
the EPF Processing Port Enabled state.
Note: I/O command processing in all of the PLS states, other than the PLS Not Ready state, complies with
atomic operation requirements for power fail, if any, as specified in the appropriate I/O Command Set
specification.
The conditions which trigger state transitions are described in the following sections. If a transition between
two states can be caused by any one of multiple conditions, then those conditions are shown in a bullet list
with an “or” (e.g., the transition from the FQ Processing state to the PLS Not Ready state, refer to Figure
664). If a transition is caused by multiple conditions which all must occur, then those conditions are shown
as a single-item bullet list with an “and” (e.g., any of the transitions from the PLS Ready state, refer to Figure
663).
8.2.5.1.1 PLS Not Ready State
In the PLS Not Ready state, the controller is not performing Power Loss Signaling processing. The controller
enters the PLS Not Ready state following any Controller Level Reset or if the CSTS.SHST field is not
cleared to 00b (i.e., the controller is in the process of shutting down or has completed shutdown).
Transitions out of this state are defined in Figure 662.
Figure 662: PLS Not Ready State Transition Conditions
State Transitions Transition Condition Starting Ending
PLS Not Ready PLS Ready • CSTS.SHST field is 00b.
8.2.5.1.2 PLS Ready State
In the PLS Ready state, the controller is not performing Power Loss Signaling processing and is not
performing shutdown processing. The controller enters the PLS Ready state when the CSTS.SHST field is
00b
Transitions out of this state are defined in Figure 663. If the controller is in this state and the CSTS.RDY bit
is cleared to ‘0’, then it is implementation specific whether the controller responds to a transition of the PLN
variable from Deasserted to Asserted.
