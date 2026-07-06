---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 523
headings: ["8.1.8.2.2 Directive Send", "8.1.8.2.2.1 Enable Directive (Directive Operation 01h)"]
---

# Source page 523

NVM Express® Base Specification, Revision 2.1

501
Figure 600: Identify Directive – Return Parameters Data Structure
Bytes Bits Description
95:64
255:16 Reserved
15
Vendor Specific Directive  (VSDIRCLR): If the Vendor Specific Directive is supported,
then this bit is:
• set to ‘1’ to indicate that the host specified Data Placement Directive state is
preserved across Controller Level Resets; or
• cleared to ‘0’ to indicate that the host specified Data Placement Directive state is
not preserved across Controller Level Resets.
If the Vendor Specific Directive is not supported, then this bit shall be cleared to ‘0’.
14:03 Reserved
02
Data Placement Directive  (DPDIRCLR): If the Data Placement Directive is supported,
then this bit shall be set to ‘1’ to indicate that the host specified Data Placement Directive
state is preserved across Controller Level Resets. If the Data Placement Directive is not
supported, then this bit shall be cleared to ‘0’.
01 Streams Directive (SDIRCLR): This bit shall be cleared to ‘0’ to indicate that the Streams
Directive state is not preserved across Controller Level Resets.
00 Identify Directive (IDIRCLR): This bit shall be set cleared to ‘0’ as the host is not able to
change the state of the Identify Directive.
4095:96 n/a Reserved
8.1.8.2.2 Directive Send
This section defines operations used with the Directive Send command for the Identify Directive.
 Enable Directive (Directive Operation 01h)
The Enable Directive operation is used to enable a specific Directive for use within a namespace by all
controllers that are associated with the same Host Identifier. The DSPEC field in command Dword 11 is not
used for this operation. The Identify Directive is always enabled. The enable state of each Directive on each
shared namespace attached to enabled controllers associated with the same non-zero Host Identifier is the
same. If the Directive is not the Data Placement Directive and  an NSID value of FFFFFFFFh is specified,
then the Enable Directive operation applies to the NVM subsystem (i.e., all namespaces and all controllers
associated with the NVM subsystem). If the Directive is the Data Placement Directive and an NSID value
of FFFFFFFFh is specified, then the controller shall abort the command with a status code of Invalid
Namespace or Format.
On a Controller Level Reset:
• all Directives other than the Identify Directive that have the Directive Persistent Across Controller
Level Resets bit cleared to ‘0’ are disabled for that controller; and
• if there is an enabled controller associated with the Host Identifier for the controller that was reset,
then for namespaces attached to enabled controllers associated with that Host Identifier, Directives
are not disabled.
If a host sets the Host Identifier of a controller to the same non -zero Host Identifier as one or more other
controllers in the NVM subsystem, then setting that Host Identifier shall result in each shared namespace
attached to that controller having the sa me enable state for each Directive as the enable state for each
Directive for that namespace attached to other controllers associated with that Host Identifier.
If a host enables a controller that has the same non-zero Host Identifier as one or more other controllers in
the NVM subsystem, then enabling that controller shall result in each shared namespace attached to that
controller having the same enable state fo r each Directive as the enable state for each Directive for that
namespace attached to other controllers associated with that Host Identifier.
For all controllers in an NVM subsystem that have the same non-zero Host Identifier, if a host changes the
enable state of any Directive for a shared namespace attached to a controller by a means other than a
