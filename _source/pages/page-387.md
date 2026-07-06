---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 387
headings: ["5.1.22 Sanitize command"]
---

# Source page 387

NVM Express® Base Specification, Revision 2.1

365
Figure 370: Namespace Management – Command Specific Status Values
Value Definition
c) the specified ANAGRPID is not supported by the controller processing the command (e.g.,
the specified value exceeds ANAGRPMAX (refer to Figure 312)).
If the host specified a non -zero ANAGRPID, retrying the command with the ANAGRPID field cleared
to 0h may succeed.
29h I/O Command Set Not Supported:  The I/O Command Set specified for a create operation is not
supported by the controller.
Dword 0 of the completion queue entry contains the Namespace Identifier created. The definition of Dword
0 of the completion queue entry is in Figure 371.
Figure 371: Namespace Management – Completion Queue Entry Dword 0
Bits Description
31:00 Namespace Identifier (NSID): This field specifies the namespace identifier created in a Create
operation. This field is reserved for all other operations.
 Sanitize command
The Sanitize command is used to start a sanitize operation that targets the NVM subsystem or to recover
from a previously failed sanitize operation that targeted the NVM subsystem.  The sanitize operation types
that may be supported are Block Erase, Crypto Erase, and Overwrite.
A sanitize operation consists of:
• sanitize processing (refer to section 8.1.24), which may include:
o deallocation of all media allocated for user data; and
o additional media modification;
• optional verification of media allocated for user data; and
• post-verification deallocation of all media allocated for user data following media verification, if any,
as described in section 8.1.24.
All sanitize operations are performed in the background (i.e., completion of the Sanitize command that
starts a sanitize operation does not  indicate completion of that sanitize operation). Refer to section 8.1.24
for details on the sanitize operation.
If the NVM subsystem supports multiple domains and the Sanitize command is not able to start a sanitize
operation as a result of the NVM subsystem being divided (refer to section 3.2.5), then the controller shall
abort the Sanitize  command shall be aborted with a status code of Asymmetric Access Inaccessible or
Asymmetric Access Persistent Loss.
The Sanitize command shall not be supported by Exported NVM Subsystems (refer to section 8.1.24.5).
The Sanitize Capabilities (SANICAP) field  in the Identify Controller data structure (refer to Figure 312)
indicates:
a) the sanitize operation types supported;
b) whether setting the No-Deallocate After Sanitize (NDAS) bit (refer to Figure 372) causes media to
be modified as part of sanitize processing;
c) whether the controller inhibits the functionality of the No -Deallocation After Sanitize bit in the
Sanitize command; and
d) whether the controller supports the Media Verification state and the Post -Verification Deallocation
state (refer to the Verification Support (VERS) bit in the SANICAP field in Figure 312).
If a Sanitize command specifies an unsupported value in the SANACT field (refer to Figure 372), then the
controller shall abort the command with a status code of Invalid Field in Command.
