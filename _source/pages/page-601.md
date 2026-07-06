---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 601
headings: ["8.1.24.3.7 Post-Verification Deallocation State"]
---

# Source page 601

NVM Express® Base Specification, Revision 2.1

579
Figure 654: Media Verification State Transition Conditions
State Transition
Transition Condition Starting Ending Label 1
Media
Verification
Post-
Verification
Deallocation
G
Either:
a) any controller in the NVM subsystem performs an Exit Media
Verification State action;
b) an NVM Subsystem Reset occurs in any domain in the NVM
subsystem;
c) a Controller Level Reset caused by a transport -specific reset
type (refer to the applicable NVMe Transport specification) of
any controller in the NVM subsystem occurs; or
d) a change in the composition of the NVM subsystem prevents
media verification.
Notes:
1. Refer to Figure 648.
Transition Media Verification:Post-Verification Deallocation:
The controller shall:
a) clear the Sanitize Progress (SPROG) field to 0h;
b) set the Media Verification Canceled (MVCNCLD) bit to ‘1’, if any controller in the NVM subsystem
processes a Controller Level Reset caused by:
• an NVM Subsystem Reset; or
• a transport-specific reset type (refer to the applicable NVMe Transport specification), if any;
c) set the Media Verification Canceled (MVCNCLD) bit to ‘1’, if a change in the composition of the
NVM subsystem prevents media verification; and
d) complete the Sanitize command with a status code of Successful Completion, if any controller in
the NVM subsystem performs an Exit Media Verification State action.
8.1.24.3.7 Post-Verification Deallocation State
In this state:
a) the controller shall deallocate all media allocated for user data in the sanitization target;
b) the Sanitize Progress (SPROG) field shall be set as described in Figure 291; and
c) all controllers and Management Endpoints in the NVM subsystem shall process commands as
described in section 8.1.24.4.
Figure 655: Post-Verification Deallocation state Transition Conditions
State Transition
Transition Condition Starting Ending Label 1
Post-
Verification
Deallocation
Idle H The controller completes deallocation of all media allocated for user
data.
Restricted
Failure I1
The sanitize operation was started by a Sanitize command specifying
the AUSE bit cleared to ‘0’ (i.e., restricted completion mode), and a
failure occurs during deallocation of all media allocated for user data.
Unrestricted
Failure I2
The sanitize operation was started by a Sanitize command specifying
the AUSE bit set to ‘1’ (i.e., unrestricted completion mode), and a failure
occurs during deallocation of all media allocated for user data.
Notes:
1. Refer to Figure 648.
