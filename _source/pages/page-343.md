---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 343
headings: []
---

# Source page 343

NVM Express® Base Specification, Revision 2.1

321
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
524 M M R
Format NVM Attributes (FNA): This field indicates attributes for the Format NVM
command.
Bits Description
7:4 Reserved
3
Format NVM Broadcast Support (FNVMBS): This bit indicates whether the
Format NVM command supports an NSID value set to FFFFFFFFh. If this bit
is set to ‘1’, then the Format NVM command does not support an NSID value
set to FFFFFFFFh. If this bit is cleared to ‘0’, then the Format NVM command
supports an NSID value set to FFFFFFFFh.
2
Cryptographic Erase Supported (CRYES):  This bit indicates whether
cryptographic erase is supported as part of the secure erase functionality. If
this bit is set to ‘1’, then cryptographic erase is supported. If this bit is cleared
to ‘0’, then cryptographic erase is not supported.
1
Secure Erase Namespace Scope (SENS):  This bit indicates whether
secure erase functionality applies to all namespaces in the NVM subsystem
or is specific to a particular namespace. If this bit is set to ’1’, then any secure
erase performed as part of a format operation results in a secure erase of all
namespaces in the NVM subsystem. If this bit is  cleared to ‘0’, then any
secure erase performed as part of a format results in a secure erase of the
particular specified namespace. If the FNVMBS bit is set to ‘ 1’, then this bit
shall be cleared to ‘0’.
0
Format Namespace Scope (FNS):  This bit indicates whether the format
operation (excluding secure erase) applies to all namespaces in the NVM
subsystem or is specific to a particular namespace. If this bit is set to ‘1’, then
all namespaces in the NVM subsystem shall be configured with the same
attributes and a format (excluding secure erase) of any namespace results
in a format of all namespaces in the NVM subsystem. If this bit is cleared to
‘0’, then the controller supports format on a per namespace basis. If the
FNVMBS bit is set to ‘1’, then this bit shall be cleared to ‘0’.
