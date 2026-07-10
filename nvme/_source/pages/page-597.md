---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 597
headings: ["8.1.24.3.3 Restricted Failure State"]
---

# Source page 597

NVM Express® Base Specification, Revision 2.1

575
Figure 650: Restricted Processing State Transition Conditions
State Transition
Transition Condition Starting Ending Label 1
Restricted
Processing
Idle C1
Sanitize processing completes successfully and the EMVS bit was:
a) cleared to ‘0’ in the Sanitize command that started the sanitize
operation; or
b) set to ‘1’ in the Sanitize command that started the sanitize
operation, the MVCNCLD bit is set to ‘1’, and deallocation of all
media allocated for user data completes successfully.
Restricted
Failure D1 Sanitize processing fails.
Media
Verification F1
The EMVS bit was set to ‘1’ in the Sanitize command that started the
sanitize operation, the sanitize processing completes successfully, and
the MVCNCLD bit is cleared to ‘0’.
Notes:
1. Refer to Figure 648.
Transition Restricted Processing:Idle:
The controller shall:
a) report the Sanitize Operation Completed asynchronous event or the Sanitize Operation Completed
With Unexpected Deallocation asynchronous event as described in section 8.1.24.2.
Transition Restricted Processing:Restricted Failure:
The controller shall:
a) set the FAILS field to 1h (i.e., Restricted Processing state); and
b) report the Sanitize Operation Completed asynchronous event or the Sanitize Operation Completed
With Unexpected Deallocation asynchronous event as described in section 8.1.24.2.
Transition Restricted Processing:Media Verification:
The controller shall:
a) report the Sanitize Operation Entered Media Verification State asynchronous event as described
in section 8.1.24.2.
8.1.24.3.3 Restricted Failure State
This state is entered if sanitize processing was performed in the Restricted Processing state and:
a) sanitize processing failed; or
b) a failure occurred during deallocation of media allocated for user data in the Post-Verification
Deallocation state.
In this state:
a) all controllers and Management Endpoints in the NVM subsystem shall process commands as
described in section 8.1.24.4;
b) failure recovery requires the host to issue a subsequent Sanitize command specifying the AUSE
bit cleared to ‘0’ (i.e., restricted completion mode);
c) all controllers in the NVM subsystem shall abort a Sanitize command specifying:
• the SANACT field set to 001b (i.e., Exit Failure Mode), with a status code of Sanitize Failed;
• the SANACT field set to 101b (i.e., Exit Media Verification State), with a status code of Invalid
Field in Command; or
• the USE bit set to ‘1’ (i.e., unrestricted completion mode), with a status code of Sanitize Failed;
