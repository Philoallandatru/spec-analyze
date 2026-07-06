---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 594
headings: ["8.1.24.3 Sanitize Operation State Machine"]
---

# Source page 594

NVM Express® Base Specification, Revision 2.1

572
• support the Sanitize Status log page;
• support the Sanitize Operation Completed asynchronous event;
• support the Sanitize Operation Completed With Unexpected Deallocation asynchronous event, if
the Sanitize Config feature is supported;
• support the Exit Failure Mode action for a Sanitize command;
• support at least one of the following sanitize operation types: Block Erase, Overwrite, or Crypto
Erase;
• support the same set of sanitize operation types;
• indicate the supported sanitize operation types in the Sanitize Capabilities field in the Identify
Controller data structure; and
• if the Verification Support (VERS) bit is set to ‘1’ in the Identify Controller data structure (refer to
Figure 312), support:
o the FAILS field in the Sanitize Status log page (refer to section 5.1.12.1.33);
o the SANS field in the Sanitize Status log page;
o the Media Verification state;
o the Post-Verification Deallocation state; and
o the Sanitize Operation Entered Media Verification State asynchronous event.
The Sanitize Config Feature Identifier (refer to section 5.1.25.1.15) contains the No-Deallocate Response
Mode (NODRM) bit that specifies the response of the controller to a Sanitize command specifying the No -
Deallocate After Sanitize (NDAS) bit (refer to Figure 372) set to ‘1’ if the No -Deallocate Inhibited bit is set
to ‘1’ in the Sanitize Capabilities field of the Identify Controller data structure (refer to Figure 312). In the
No-Deallocate Error Response Mode, the controller aborts such Sanitize commands with a status code of
Invalid Field in Command. In the No -Deallocate Warning Response Mode, the controller processes such
Sanitize commands, and if a resulting saniti ze operation is completed successfully, then the SOS field is
set to 100b (i.e., Sanitized Unexpected Deallocate) in the Sanitize Status log page (refer to Figure 291).
 Sanitize Operation State Machine
The Sanitize Operation State Machine (refer to Figure 648) defines the state of sanitization of a sanitization
target. The label on each transition begins with a letter and may include a number. The letter indicates the
condition causing that transition, as described in the section for each state. The number dif ferentiates
between different transitions that have the same transition condition.
