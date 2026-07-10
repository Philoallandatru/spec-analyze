---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 595
headings: ["8.1.24.3.1 Idle State"]
---

# Source page 595

NVM Express® Base Specification, Revision 2.1

573
Figure 648: Sanitize Operation State Machine
C Restricted
Failure
Unrestricted
Failure
A1
B1
D1
CD2 H
G
C2
C1
A2
A3
B2
E Media
Verification
Restricted
Processing
Unrestricted
Processing
Idle
F1
F2
I2
I1
Post-
Verification
Deallocation

In the state transitions described in this section, asynchronous events are reported as described in section
8.1.24.2. In Completion Queue Entry Dword 0 (refer to Figure 147) for the Asynchronous Event Request
command:
a) the Log Page Identifier field shall be set to 81h (i.e., Sanitize Status log page);
b) the Asynchronous Event Information field (refer to section 5.1.2.1) shall be set to:
• 01h (i.e., Sanitize Operation Completed);
• 02h (i.e., Sanitize Operation Completed With Unexpected Deallocation); or
• 03h (i.e., Sanitize Operation Entered Media Verification State);
and
c) the Asynchronous Event Type field shall be set to 110b (i.e., I/O Command specific status).
In each state transition described in this section, the controller shall set the Sanitize State (SANS) field to
the value corresponding to the state being entered.
8.1.24.3.1 Idle State
In this state, no sanitize operation is in process. This state applies in the following cases:
a) no sanitize operation has ever been performed on the NVM subsystem since the NVM subsystem
was manufactured;
b) the most recent sanitize operation on the NVM subsystem was successful; and
c) the most recent sanitize operation failed in unrestricted completion mode (i.e., the Sanitize
command specified the AUSE bit set to ‘1’) and then the Sanitize Operation State Machine
transitioned from the Unrestricted Failure state to the Idle state when a ny controller in the NVM
subsystem performed an Exit Failure Mode action.
In this state, any controller in the NVM subsystem processing a Sanitize command specifying the Sanitize
Action field set to 001b (i.e., Exit Failure Mode) shall not be considered an error.
In this state, all controllers in the NVM subsystem are permitted to resume any power management that
was suspended by any prior sanitize operation.
