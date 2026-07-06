---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 596
headings: ["8.1.24.3.2 Restricted Processing State"]
---

# Source page 596

NVM Express® Base Specification, Revision 2.1

574
Figure 649: Idle State Transition Conditions
State Transition
Transition Condition Starting Ending Label 1
Idle
Restricted
Processing A1 The controller starts a sanitize operation in restricted completion mode (i.e.,
the Sanitize command specified the AUSE bit cleared to ‘0’).
Unrestricted
Processing B1 The controller starts a sanitize operation in unrestricted completion mode
(i.e., the Sanitize command specified the AUSE bit set to ‘1’).
Notes:
1. Refer to Figure 648.
Transition Idle:Restricted Processing:
The controller shall clear the:
a) Sanitize Progress (SPROG) field to 0h; and
b) Media Verification Canceled (MVCNCLD) bit to ‘0’.
Transition Idle:Unrestricted Processing:
The controller shall clear the:
a) Sanitize Progress (SPROG) field to 0h; and
b) Media Verification Canceled (MVCNCLD) bit to ‘0’.
8.1.24.3.2 Restricted Processing State
In this state, if the sanitize operation fails, then the sanitization target transitions to the Restricted Failure
state.
In this state:
a) the controller shall set the Sanitize Progress (SPROG) field as described in Figure 291;
b) the controller shall perform additional media modification, if required, as described in section
8.1.24.1;
c) the controller should deallocate all media allocated for user data, if permitted, as described in
section 8.1.24.1;
d) if a change in the composition of the NVM subsystem occurs then the MVCNCLD bit shall be set
to ‘1’;
e) if a Controller Level Reset of any controller in the NVM subsystem occurs that is caused by:
• a transport-specific reset type (refer to the applicable NVMe Transport specification); or
• an NVM Subsystem Reset,
then the MVCNCLD bit shall be set to ‘1; and
f) all controllers and Management Endpoints in the NVM subsystem shall process commands as
described in section 8.1.24.4.
