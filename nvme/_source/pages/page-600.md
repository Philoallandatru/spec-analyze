---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 600
headings: ["8.1.24.3.6 Media Verification State"]
---

# Source page 600

NVM Express® Base Specification, Revision 2.1

578
with a status code of Sanitize Failed; and
d) the Persistent Memory Region shall behave as described in section 8.1.24.4.
Figure 653: Unrestricted Failure State Transition Conditions
State Transition
Transition Condition Starting Ending Label 1
Unrestricted
Failure
Restricted
Processing A3 The controller starts a sanitize operation in restricted completion mode
(i.e., the Sanitize command specified the AUSE bit cleared to ‘0’).
Unrestricted
Processing B2 The controller starts a sanitize operation in unrestricted completion mode
(i.e., the Sanitize command specified the AUSE bit set to ‘1’).
Idle E Any controller in the NVM subsystem performs an Exit Failure Mode
action.
Notes:
1. Refer to Figure 648.
Transition Unrestricted Failure:Restricted Processing:
The controller shall start a sanitize operation in the Restricted Processing state. The controller shall:
a) clear the Sanitize Progress (SPROG) field to 0h; and
b) clear the Media Verification Canceled (MVCNCLD) bit to ‘0’.
Transition Unrestricted Failure:Unrestricted Processing:
The controller shall start a sanitize operation in the Unrestricted Processing state. The controller shall:
a) clear the Sanitize Progress (SPROG) field to 0h; and
b) clear the Media Verification Canceled (MVCNCLD) bit to ‘0’.
Transition Unrestricted Failure:Idle:
If any controller in the NVM subsystem performs an Exit Failure Mode action, then the controller  shall
recover from the sanitization failure by transitioning the Sanitize Operation State Machine to the Idle state
and shall complete the Sanitize command that specified the Exit Failure Mode action with a status code of
Successful Completion.
8.1.24.3.6 Media Verification State
In this state, the sanitize processing completed successfully, and all media allocated for user data in the
sanitization target is readable by the host for purposes of verifying sanitization.
In this state:
a) the Sanitize Operation State Machine shall transition to the Post -Verification Deallocation state if
any controller in the NVM subsystem performs an Exit Media Verification State action;
b) all controllers in the NVM subsystem shall abort a Sanitize command specifying the SANACT field
not set to 101b (i.e., Exit Media Verification State) with a status code of Invalid Field in Command;
and
c) all controllers and Management Endpoints in the NVM subsystem shall process commands as
described in section 8.1.24.4, with exceptions as described in the appropriate I/O command set
specification (e.g., the Read command in the NVM Command Set is processed as described in the
Media Verification section of the NVM Command Set Specification).
