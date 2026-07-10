---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 218
headings: []
---

# Source page 218

NVM Express® Base Specification, Revision 2.1

196
As part of the Format NVM command, the host requests a format operation and may request a secure
erase of the contents of the NVM (refer to the SES field in Figure 189). There are two types of secure erase.
The User Data Erase erases all user content present in the NVM subsystem. The Cryptographic Erase
erases all user content present in the NVM subsystem by deleting the encryption key with which the user
data was previously encrypted.
The scope of the format operation and the scope of the format with secure erase depend on the attributes
that the controller supports for the Format NVM command and the Namespace Identifier specified in the
command as described in Figure 188. The type of secure erase, if applicable, is based on the setting of the
Secure Erase Settings field in Command Dword 10 as defined in Figure 189.
The scope of the entire Format NVM command is determined by the value of the SES field (refer to Figure
188) and the setting of either the Format Namespace Scope (FNS) bit or the Secure Erase Namespace
Scope (SENS) bit (refer to Figure 312) as follows:
a) If the Format NVM command does not specify Secure Erase (i.e., the SES field is cleared to 000b),
then the scope of the Format NVM command is indicated by the value of FN S bit and the value of
SENS bit is not applicable to this command.
b) If the Format NVM command specifies Secure Erase (i.e., the SES field is set to a non-zero value),
then the scope of the Format NVM command is indicated by the value of SENS bit and the value
of FNS bit is not applicable to this command.
Figure 188: Format NVM – Operation Scope
SES SENS Bit FNS Bit NSID Format Operation
000b (i.e.,
not a
secure
erase)
0
N/A
FFFFFFFFh1 All namespaces attached to the controller.
Other namespaces are not affected.
0 Any allocated value
(refer to section 3.2.1.3)
The specified namespace. Other
namespaces are not affected.
1
Any allocated value
(refer to section 3.2.1.3)
or FFFFFFFFh
All namespaces that exist in the NVM
subsystem.
001b or
010b (i.e.,
secure
erase)
N/A
0 FFFFFFFFh1 All namespaces attached to the controller.
Other namespaces are not affected.
0 Any allocated value
(refer to section 3.2.1.3)
The specified namespace. Other
namespaces are not affected.
1
Any allocated value
(refer to section 3.2.1.3)
or FFFFFFFFh
All namespaces that exist in the NVM
subsystem.
All others
The controller shall abort the command
with a status code of Invalid Field in
Command
Notes:
1. If the Format NVM Broadcast Support (FNVMBS) bit in the FNA field is set to ‘1’, then this value is not supported,
and the command is aborted as described in this section.
If the NVM subsystem supports multiple domains and the Format NVM command is not able to format the
specified namespaces as a result of the NVM subsystem being divided (refer to section 3.2.5), then the
Format NVM command shall be aborted with a status code of Asymmetric Access Inaccessible or
Asymmetric Access Persistent Loss.
The Format NVM command may be aborted with a status code defined in this specification under
circumstances defined by a security specification (e.g., invalid security state as specified in TCG Storage
Interface Interactions specification). If there are I/O  commands being processed for a namespace, then a
Format NVM command that is submitted affecting that namespace may be aborted; if aborted, then a status
code of Command Sequence Error shall be returned. If a Format NVM command is in progress, then an
I/O command that is submitted for any namespace affected by that Format NVM command may be aborted;
