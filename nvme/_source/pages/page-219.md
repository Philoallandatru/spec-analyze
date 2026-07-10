---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 219
headings: []
---

# Source page 219

NVM Express® Base Specification, Revision 2.1

197
if aborted, then a status code of Format in Progress shall be returned.  Refer to section  5 for further
information about restrictions on Admin commands during Format NVM.
For a Format NVM command with the NSID field set to FFFFFFFFh that specifies secure erase:
a) if the Secure Erase Namespace Scope (SENS) bit is set to ‘1’ in the FNA field (refer to Figure 312)
and there are no namespaces in the NVM subsystem, then that Format NVM command shall
complete without error; and
b) if the SENS bit is cleared to ‘0’ in the FNA field and there are no attached namespaces, then that
Format NVM command shall complete without error.
For a Format NVM command with an NSID field set to FFFFFFFFh that does not specify a secure erase:
a) if the Format Namespace Scope (FNS) bit is set to ‘1’ in the FNA field and there are no namespaces
in the NVM subsystem, then that Format NVM command shall complete without error; and
b) if the FNS bit is cleared to ‘0’ in the FNA field and there are no attached namespaces, then that
Format NVM command shall complete without error.
If a host does not set the LBA Format Extension Enable (LBAFEE) field to 1h in the Host Behavior Support
feature (refer to section 5.1.25.1.14), then the 0h value of the LBAFEE field disables any I/O Command Set
specific format that requires the LBAFEE field to be set to 1h (refer to the applicable I/O Command Set
specification). If a Format NVM command specifies a format that is disabled (e.g., the LBA FEE field is
cleared to 0h), then the controller shall abort that Format NVM command with a status code of Invalid
Namespace or Format.
If the format operation scope (refer to Figure 188) for a Format NVM command includes any namespace
that is write protected (refer to section 8.1.16), then the controller aborts that Format NVM command with
a status code of Namespace is Write Protected.
If the Format NVM Broadcast Support (FNVMBS) bit in the FNA field is set to ‘1’ and a Format NVM
command has the NSID field set to FFFFFFFFh, then the controller shall abort the command with a status
code of Invalid Field in Command.
After successful completion of a Format NVM command, the settings specified in the Format NVM
command (e.g., PI, MSET, LBAF) are reported as part of the Identify Namespace data structure s (refer to
section 1.5.49).
The Format NVM command uses the Command Dword 10 field. All other command specific fields are
reserved.
Figure 189: Format NVM – Command Dword 10
Bits Description
31:14 Reserved
13:12
LBA Format Upper (LBAFU): This field specifies the most significant 2 bits of the Format Index of the
User Data Format to apply to the NVM media. This corresponds to the User Data Formats indicated in
the Identify command, refer to the Identify Namespace data structure and the LBA Format data structure
in the applicable I/O Command Set specification . If an unsupported User Data Format is selected, the
controller shall abort the command with a status code of Invalid Format.
This field shall be ignored by the controller if the LBA Format Extension Enable (LBAFEE) field is cleared
to 0h in the Host Behavior Support feature (refer to section 5.1.25.1.14).
NOTE: This field applies to all User Data Formats . The original name has been retained for historical
continuity.
