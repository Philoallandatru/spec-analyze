---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 384
headings: ["5.1.20.1 Command Completion"]
---

# Source page 384

NVM Express® Base Specification, Revision 2.1

362
o For each controller specified in the controller list, if attaching the namespace to that I/O
controller causes the sum of the number of namespaces attached to each I/O controller in
the Domain to be greater than the value specified in the MAXDNA field, then the controller
shall abort the command with a status code of Namespace Attachment Limit Exceeded;
and
• For each I/O controller specified in the controller list, if the Maximum I/O Controller Namespace
Attachments (MAXCNA) field in the Identify Controller data structure for that controller is non-zero,
then:
o If attaching the namespace to that I/O controller causes that I/O controller to have the
number of attached namespaces to be greater than the value specified in the MAXCNA
field, then the controller shall abort the command with a status code of Namespace
Attachment Limit Exceeded.
If an attempt is made to attach a namespace to a controller that does not support the corresponding I/O
Command Set, then the command shall be aborted with a status code of I/O Command Set Not Supported.
If an attempt is made to attach a namespace to a controller that supports the corresponding I/O Command
Set and the corresponding I/O Command Set is not enabled by the I/O Command Set profile feature, then
the command shall be aborted with a status code of I/O Command Set Not Enabled.
Figure 363: Namespace Attachment – Data Pointer
Bits Description
127:00
Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field. If using PRPs, this field shall not be a pointer to a PRP List as the data buffer may not cross
more than one page boundary.

Figure 364: Namespace Attachment – Command Dword 10
Bits Description
31:04 Reserved
03:00
Select (SEL): This field selects the type of attachment to perform.
Value Definition
0h Controller Attach
1h Controller Detach
2h to Fh Reserved

 Command Completion
When the command is completed, the controller posts a completion queue entry to the Admin Completion
Queue indicating the status for the command.
Command specific status values associated with the Namespace Attachment command are defined in
Figure 365. For failures, the byte offset of the first failing entry is reported in the Command Specific
Information field of the Error Information Log Entry. The controller does not process further entries in the
Controller List after an error is encountered.
Figure 365: Namespace Attachment – Command Specific Status Values
Value Definition
18h Namespace Already Attached: The controller is already attached to the specified namespace.
19h Namespace Is Private: The controller is not attached to the namespace. The request to attach the controller
could not be completed because the namespace is private and is already attached to one controller.
1Ah Namespace Not Attached: The controller is not attached to the namespace. The request to detach the
controller could not be completed.
