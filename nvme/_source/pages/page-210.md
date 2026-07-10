---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 210
headings: ["5.1.5 Device Self-test command"]
---

# Source page 210

NVM Express® Base Specification, Revision 2.1

188
Figure 168: Controller Data Queue – Completion Queue Entry Dword 0
Bits Description
31:16 Reserved
15:00 Controller Data Queue Identifier (CDQID): This field indicates the identifier for the Controller Data
Queue created (refer to section 8.1.6).
Controller Data Queue command specific status values (i.e., SCT field set to 1h) are shown in Figure 169.
Figure 169: Controller Data Queue – Command Specific Status Values
Value Definition
1Fh Invalid Controller Identifier: The Controller Identifier field contains an invalid value.
37h Invalid Controller Data Queue: This error indicates that the specified Controller Data Queue Identifier
is invalid for the controller processing the command.
 Device Self-test command
The Device Self -test command is used to start a device self -test operation or abort a device self -test
operation (refer to section 8.1.7). The Device Self-test command is used specifically to:
a) start a short device self-test operation;
b) start an extended device self-test operation;
c) start a Host-Initiated Refresh operation;
d) start a vendor specific device self-test operation; or
e) abort a device self-test operation already in process.
A Host-Initiated Refresh operation is a device self-test operation.
The device self -test operation is performed by the controller that the Device Self -test command was
submitted to. The Namespace Identifier field controls which namespaces are included in the device self -
test operation as specified in Figure 170. For a Host-Initiated Refresh operation, the Namespace Identifier
field shall be ignored by the controller.
Figure 170: Device Self-test Namespace Test Action
NSID Value Description
00000000h Specifies that the device self -test operation shall not include any namespaces, and only the
controller is included as part of the device self-test operation.
00000001h to
FFFFFFFEh
Specifies that the device self -test operation shall include the specified namespace. If this field
specifies an invalid namespace ID, then the controller shall abort the command with a status
code of Invalid Namespace or Format. If this field specifies an inactive namespace ID, then the
controller shall abort the command with a status code of Invalid Field in Command.
FFFFFFFFh Specifies that the device self -test operation shall include all attached namespaces accessible
through the controller at the time the device self-test operation is started.
The Device Self-test command uses the Command Dword 10 field  and the Command Dword 15 field . All
other command specific fields are reserved.
Figure 171: Device Self-test – Command Dword 10
Bits Description
31:04 Reserved
