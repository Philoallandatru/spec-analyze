---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 417
headings: ["5.1.25.1.25 Host Management Agent Address (Feature Identifier 79h)", "5.1.25.1.26 Host Metadata (Feature Identifier 7Dh), (Feature Identifier 7Eh), (Feature Identifier 7Fh)"]
---

# Source page 417

NVM Express® Base Specification, Revision 2.1

395
The Command and Feature Lockdown log page (refer to section 5.1.12.1.20), if supported, should indicate
that this Feature is supported to be prohibited from execution.
5.1.25.1.25 Host Management Agent Address (Feature Identifier 79h)
This Feature configures a URI (refer to RFC 3986) containing the address of a management agent provided
by host software for management of the NVM subsystem.
If a Set Features command is issued for this Feature, the data structure specified in Figure 429 is transferred
in the data buffer for that command.
If a Get Features command is issued for this Feature, the data structure specified in Figure 429 is returned
in the data buffer for that command.
Figure 429: Host Management Agent Address
Bytes Description
03:00 Reserved
511:04 Host Management Agent Address (HMAA): A URI (refer to RFC 3986), containing the address of a
management agent residing in host software.
The Command and Feature Lockdown log page (refer to section 5.1.12.1.20), if supported, should indicate
that this Feature is supported to be prohibited from execution.
5.1.25.1.26 Host Metadata (Feature Identifier 7Dh), (Feature Identifier 7Eh), (Feature Identifier 7Fh)
The Host Metadata features are the Enhanced Controller Metadata feature (Feature Identifier 7Dh), the
Controller Metadata feature (Feature Identifier 7Eh), and the Namespace Metadata feature (Feature
Identifier 7Fh).
If a Get Features command specifying one of the Host Metadata features with the SEL field set to 011b
(i.e., Supported Capabilities) is submitted, then the Saveable bit in Dword 0 of the corresponding completion
queue entry shall be cleared to ‘0’ (i.e., r efer to section 4.4), and the Changeable bit in Dword 0 of the
corresponding completion queue entry shall be set to ‘1’.
If a Get Features command specifying one of the Host Metadata features, the controller shall perform
additional actions specified in Figure 430.
Figure 430: Get Features – Command Dword 11
Bits Description
31:01 Reserved
