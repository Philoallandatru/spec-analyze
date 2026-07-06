---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 555
headings: ["8.1.16.2 Namespace Write Protection – Command Interactions"]
---

# Source page 555

NVM Express® Base Specification, Revision 2.1

533
o setting the Permanent Write Protect Support bit  to ‘1’ in the NWPC field, if the Permanent
Write Protect state is supported;
and
• support the Namespace Write Protection Config feature (refer to section 5.1.25.1.31).
If the controller supports the Write Protect Until Power Cycle state or the Permanent Write Protect state ,
then the controller shall support the Write Protection Control field in the RPMB Device Configuration Block
data structure (refer to section 8.1.21).
The controller shall not set the All Media Read -Only bit to ‘1’ in the Critical Warning field (refer to Figure
206) if the read-only condition on the media is a result of a change in the namespace write protection state
as defined by the Namespace Write Protection State Machine (refer to Figure 622), or due to any
autonomous namespace write protection state transitions (e.g., power cycle). Host software may check the
current namespace write protection state of a namespace using the Get Features command with the
Namespace Write Protection Config Feature Identifier.
If any controller in the NVM subsystem supports Namespace Write Protection, then the write protection
state of a namespace shall be enforced by any controller to which that namespace is attached.
 Namespace Write Protection – Command Interactions
Unless otherwise noted, the commands listed in Figure 623 are processed normally when specifying an
NSID for a namespace that is write protected.
Figure 623: Commands Allowed when Specifying a Write Protected NSID
Admin Command Set NVM Command Set
Device Self–test Compare
Directive Send 1 Dataset Management 1
Directive Receive 3 Read
Get Features Reservation Register
Get Log Page Reservation Report
Identify Reservation Acquire
Namespace Attachment Reservation Release
Security Receive 1 Vendor Specific 1
Security Send 1 Flush 2
Set Features 1 Verify
Vendor Specific 1
Notes:
1. The controller shall fail commands if the specified action attempts to modify the non-volatile storage medium of
the specified namespace.
2. A Flush command shall complete successfully with no effect. All volatile write cache data and metadata
associated with the specified namespace is written to non-volatile storage medium as part of transitioning to the
write protected state (refer to section 5.1.25.1.31).
3. A Directive Receive command which attempts to allocate streams resources shall be aborted with a status code
of Namespace is Write Protected.
Commands not listed in Figure 623, and which meet the following conditions, shall be aborted with a status
code of Namespace Is Write Protected (refer to Figure 102):
a) Commands that specify an NSID for a namespace that is write protected;
b) Commands that specify an NSID for a namespace that is not write protected and the execution of
which would modify another namespace that is write protected (e.g., a Format NVM command);
and
