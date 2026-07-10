---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 599
headings: ["8.1.24.3.5 Unrestricted Failure State"]
---

# Source page 599

NVM Express® Base Specification, Revision 2.1

577
Figure 652: Unrestricted Processing State Transition Conditions
State Transition
Transition Condition Starting Ending Label 1
Unrestricted
Processing
Idle C2
The sanitize processing completes successfully and the EMVS bit was:
a) cleared to ‘0’ in the Sanitize command that started the sanitize
operation; or
b) set to ‘1’ in the Sanitize command that started the sanitize
operation, the MVCNCLD bit is set to ‘1’, and deallocation of all
media allocated for user data completes successfully.
Unrestricted
Failure D2 The sanitize processing fails.
Media
Verification F2
The EMVS bit was set to ‘1’ in the Sanitize command that started the
sanitize operation, the sanitize processing completes successfully, and
the MVCNCLD bit is cleared to ‘0’.
Notes:
1. Refer to Figure 648.
Transition Unrestricted Processing:Idle:
The controller shall:
a) report the Sanitize Operation Completed asynchronous event or the Sanitize Operation Completed
With Unexpected Deallocation asynchronous event as described in section 8.1.24.2.
Transition Unrestricted Processing:Unrestricted Failure:
The controller shall:
a) set the FAILS field to 3h (i.e., Unrestricted Processing state); and
b) report the Sanitize Operation Completed asynchronous event or the Sanitize Operation Completed
With Unexpected Deallocation asynchronous event as described in section 8.1.24.2.
Transition Unrestricted Processing:Media Verification:
The controller shall:
a) report the Sanitize Operation Entered Media Verification State asynchronous event as described
in section 8.1.24.2.
8.1.24.3.5 Unrestricted Failure State
This state is entered if sanitize processing was performed in the Unrestricted Processing state and:
a) sanitize processing failed; or
b) a failure occurred during deallocation of media allocated for user data in the Post -Verification
Deallocation state.
In this state:
a) all controllers and Management Endpoints in the NVM subsystem shall process commands as
described in section 8.1.24.4;
b) all controllers in the NVM subsystem shall abort a Sanitize command specifying the SANACT field
set to 101b (i.e., Exit Media Verification State) with a status code of Invalid Field in Command;
c) all controllers in the NVM subsystem shall abort a Sanitize command specifying the SANACT field
set to a value other than:
• 001b (i.e., Exit Failure Mode);
• 010b (i.e., Start a Block Erase sanitize operation);
• 011b (i.e., Start an Overwrite sanitize operation); or
• 100b (i.e., Start a Crypto Erase sanitize operation),
