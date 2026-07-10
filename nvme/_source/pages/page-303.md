---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 303
headings: []
---

# Source page 303

NVM Express® Base Specification, Revision 2.1

281
03:02
Sanitize Status (SSTAT):  This field indicates the status associated with the most recent sanitize
operation.
Bits Description
15:10 Reserved
09
Media Verification Canceled (MVCNCLD):  This bit indicates whether the Media
Verification state is canceled.
If the most recent sanitize operation was started by a Sanitize command with the EMVS
bit set to ‘1’ and during the Restricted Processing state, the Unrestricted Processing
state, or the Media Verification state:
a) a change in the composition of the NVM subsystem prevents media verification;
or
b) a Controller Level Reset of any controller in the NVM subsystem occurs that is
caused by:
• a transport -specific reset type  (refer to the applicable NVMe Transport
specification); or
• an NVM Subsystem Reset,
then this bit is set to ‘1’. Otherwise, this bit is cleared to ‘0’.
Refer to the Sanitize Operation State Machine defined in section 8.1.24.3 for conditions
that affect the setting of this bit.
08
Global Data Erased (GDE): If no user data has been written to the NVM subsystem and
no Persistent Memory Region in the NVM subsystem has been enabled:
a) since being manufactured and the NVM subsystem has never been sanitized;
or
b) since the most recent successful sanitize operation,
then this bit shall be set to ‘1’. Otherwise, this bit shall be cleared to ‘0’.
07:03
Overwrite Passes Completed (OPC): This field shall indicate the number of completed
passes if the most recent sanitize operation was an Overwrite. This field shall be cleared
to 0h if the most recent sanitize operation was not an Overwrite.
02:00
Sanitize Operation Status (SOS):  This field indicates the status of the most recent
sanitize operation as shown below. Sanitize states are described in section 8.1.24.3.
Value State
000b
Sanitize Never Started:  If a sanitize operation has never been
started in the NVM subsystem, then this value shall be reported .
Otherwise, this value shall not be reported.
001b
Sanitized: If:
a) the most recent sanitize operation completed
successfully; and
b) the Sanitize Operation State Machine is in the Idle state,
then this value shall be reported. Otherwise, this value shall not be
reported.
010b
Sanitizing: If a sanitize operation is currently in progress (i.e., in
the Restricted Processing state, the Unrestricted Processing state,
the Media Verification state, or the Post -Verification Deallocation
state), then this value shall be reported. Otherwise, this value shall
not be reported.
011b
Sanitize Failed: If the most recent sanitize operation failed, then
this value shall be reported . Otherwise , this value shall not be
reported.
This value shall be reported when the Sanitize Operation State
Machine is in the Restricted Failure state or the Unrestricted Failure
state.
This value is able to be reported when the Sanitize Operation State
Machine is in the Idle state.
100b Sanitized Unexpected Deallocate: If:
