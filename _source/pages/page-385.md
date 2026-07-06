---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 385
headings: ["5.1.21 Namespace Management command"]
---

# Source page 385

NVM Express® Base Specification, Revision 2.1

363
Figure 365: Namespace Attachment – Command Specific Status Values
Value Definition
1Ch Controller List Invalid: The controller list provided is invalid or the controller list contains an Administrative
controller.
25h ANA Attach Failed: The controller is not attached to the namespace as a result of an ANA condition (e.g.,
attaching the controller would result in an ANA Persistent Loss state (refer to section 8.1.1.7)).
27h Namespace Attachment Limit Exceeded:  Attaching the namespace to a controller causes maximum
number of namespace attachments allowed to be exceeded.
29h I/O Command Set Not Supported: The request to attach the controller could not be completed due to the
I/O Command Set corresponding to the namespace is not supported by the controller.
2Ah I/O Command Set Not Enabled: The request to attach the controller could not be completed due to the I/O
Command Set corresponding to the namespace is restricted by the I/O Command Set profile feature.
 Namespace Management command
The Namespace Management command is used to manage namespaces (refer to section 8.1.15), including
create and delete operations.
Note: The controller continues to execute commands submitted to I/O Submission Queues while this
operation is in progress.
If the Namespace Management command supported, then the Namespace Attachment command (refer to
section 5.1.20) shall also be supported. The Namespace Management command shall not be supported by
controllers in an Exported NVM subsystem.
The h ost uses the Namespace Attachment command to attach or detach a namespace to or from a
controller. The create operation does not attach the namespace to a controller. As a side effect of the delete
operation, the namespace is detached from all controllers as the namespace is no longer present in the
system. It is recommended that host software detach all controllers from a namespace prior to deleting the
namespace. If the namespace is not detached from all controllers prior to being deleted, then Attac hed
Namespace Attribute Changed asynchronous events are reported as specified in section 8.1.15.2. If a
namespace is deleted, then Allocated Namespace Attribute Changed asynchronous events are reported
as specified in section 8.1.15.2.
The data structure used for the create operation is defined in Figure 369 and the CSI field specifies the I/O
Command Set specification. There is no data structure transferred for the delete operation.
The Namespace Management command uses the Data Pointer, Command Dword 10, and Command
Dword 11 fields. All other command specific fields are reserved.
The Namespace Identifier (NSID) field is used as follows for create and delete operations:
• Create: The NSID field is reserved for this operation; host software clears this field to a value of
0h. The controller shall select an available Namespace Identifier to use for the operation; or
• Delete: This field specifies the previously created namespace to delete in this operation. Specifying
a value of FFFFFFFFh is used to delete all namespaces  in the NVM subsystem . If the value of
FFFFFFFFh is specified and there are zero valid namespaces, the command completes
successfully.
Figure 366: Namespace Management – Data Pointer
Bits Description
127:00
Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field. If using PRPs, this field shall not be a pointer to a PRP List as the data buffer may not cross
more than one page boundary.
