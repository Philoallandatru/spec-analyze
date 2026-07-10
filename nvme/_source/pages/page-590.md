---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 590
headings: []
---

# Source page 590

NVM Express® Base Specification, Revision 2.1

568
Additional media modification shall be performed if the NODMMAS field is set to 10b (refer to Figure 312)
and the Sanitize command that started the sanitize operation specifies:
• the Enter Media Verification State (EMVS) bit cleared to ‘0’; and
• the No-Deallocate After Sanitize (NDAS) bit set to ‘1’.
Verification of media allocated for user data is able to be performed by the host if the Sanitize Operation
State Machine is in the Media Verification state. A Block Erase sanitize operation or Crypto Erase sanitize
operation may invalidate error correction codes on the media, causing subsequent reads to fail because of
media errors. In the Media Verification state, reads are successful regardless of such invalid error correction
codes which enables the host to perform an audit (refer to section 1.5.11) to verify that the media was
sanitized. Refer to the applicable I/O Command Set specification for details. Media verification is performed
if the Sanitize command that starts a sanitize operation specifies the EMVS bit set to ‘1’ and sanitize
processing completes successfully.
If a sanitize operation includes deallocation of all media allocated for user data, then that deallocation shall
be performed in exactly one of the following states:
• Restricted Processing state;
• Unrestricted Processing state; or
• Post-Verification Deallocation state.
If the Sanitize command (refer to Figure 372) that starts a sanitize operation specifies:
• the Enter Media Verification State (EMVS) bit cleared to ‘0’;
• the AUSE bit cleared to ‘0’; and
• the No-Deallocate After Sanitize (NDAS) bit is:
o cleared to ‘0’; or
o set to ‘1’ and the controller encounters a condition that results in unexpected deallocation
of all media allocated for user data (refer to section 5.1.25.1.15),
then deallocation of all media allocated for user data shall be performed in the Restricted Processing state.
If that deallocation fails, then sanitize processing fails.
If Media Verification state is canceled (i.e., the MVCNCLD bit is set to ‘1’) during the Restricted Processing
state, then deallocation of all media allocated for user data shall be performed in the Restricted Processing
state.
If the Sanitize command that starts a sanitize operation specifies:
• the Enter Media Verification State (EMVS) bit cleared to ‘0’;
• the AUSE bit set to ‘1’; and
• the No-Deallocate After Sanitize (NDAS) bit is:
o cleared to ‘0’; or
o set to ‘1’ and the controller encounters a condition that results in unexpected deallocation
of all media allocated for user data (refer to section 5.1.25.1.15),
then deallocation of all media allocated for user data shall be performed in the Unrestricted Processing
state. If that deallocation fails, then sanitize processing fails.
If the Media Verification state is canceled (i.e., the MVCNCLD bit is set to ‘1’) during the Unrestricted
Processing state, then deallocation of all media allocated for user data shall be performed in the
Unrestricted Processing state.
In the Post-Verification Deallocation state the controller deallocates all user data.
In the Post-Verification Deallocation state, if the controller:
• successfully completes deallocating all media allocated for user data, then the sanitization target
enters the Idle state; or
