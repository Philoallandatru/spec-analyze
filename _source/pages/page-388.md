---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 388
headings: []
---

# Source page 388

NVM Express® Base Specification, Revision 2.1

366
If the Verification Support (VERS) bit is cleared to ‘0’ and a Sanitize command specifies the Enter Media
Verification State (EMVS) bit set to ‘1’ (refer to Figure 372), then the controller shall abort the command
with a status code of Invalid Field in Command.
If the Verification Support (VERS) bit is set to ‘1’ and a Sanitize command is processed that specifies:
a) the Enter Media Verification State (EMVS) bit set to ‘1’, the SANACT field set to a value of 010b
(i.e., Block Erase) or a value of 100b (i.e., Crypto Erase), and the No -Deallocate After Sanitize
(NDAS) bit cleared to ‘0’, then successful sanitize processi ng is followed by entry to the Media
Verification state;
b) the Enter Media Verification State (EMVS) bit set to ‘1’ and:
a) the SANACT field set to 011b (i.e., Overwrite); or
b) the No-Deallocate After Sanitize (NDAS) bit set to ‘1’;
then the controller shall abort the command with a status code of Invalid Field in Command;
c) the SANACT field set to 101b (i.e., Exit Media Verification State) and the sanitization target is in
the Media Verification state, then:
• the controller does not start a new sanitize operation; and
• the sanitization target transitions from the Media Verification state to the Post -Verification
Deallocation state, in which all media allocated for user data in the NVM subsystem is
deallocated;
or
d) the SANACT field set to 101b (i.e., Exit Media Verification State) and the sanitization target is not
in the Media Verification state, then the controller shall abort the command with a status code of
Invalid Field in Command.
If any Persistent Memory Region is enabled in an NVM subsystem, then the controller shall abort any
Sanitize command with a status code of Sanitize Prohibited While Persistent Memory Region is Enabled.
If any namespace is write protected in an NVM subsystem (refer to section 8.1.16), then the controller
aborts any Sanitize command with a status code of Namespace is Write Protected.
If a firmware activation with reset is pending, then the controller shall abort any Sanitize command (refer to
section 5.1.8.1 and section 3.11).
If the Firmware Commit command that established the pending firmware activation with reset condition
returned a status code of:
a) Firmware Activation Requires Controller Level Reset;
b) Firmware Activation Requires Conventional Reset; or
c) Firmware Activation Requires NVM Subsystem Reset,
then the controller shall abort the Sanitize command with that same status code.
If the Firmware Commit command that established the pending firmware activation with reset condition
completed successfully or returned a status code other than:
a) Firmware Activation Requires Controller Level Reset;
b) Firmware Activation Requires Conventional Reset; or
c) Firmware Activation Requires NVM Subsystem Reset,
then the controller shall abort the Sanitize command with a status code of Firmware Activation Requires
Controller Level Reset.
Activation of new firmware is prohibited during a sanitize operation (refer to section 8.1.24.4).
If one or more controllers in the NVM subsystem is suspended (refer to section  5.1.17.1.1), then the
controller shall abort the command with a status code of Controller Suspended.
