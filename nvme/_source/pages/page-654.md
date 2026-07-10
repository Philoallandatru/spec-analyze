---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 654
headings: ["8.3.2.3.8.3.3 Remove Active ZoneGroup (RAZ)"]
---

# Source page 654

NVM Express® Base Specification, Revision 2.1

632
Figure 710: FZR Data for Pull Model AAZ
Bytes Description
00 Operation Type (OTYP): 9h (i.e., Add/Replace Active ZoneGroup for a pull model DDC)
07:01 Reserved
11:08 ZoneGroup Fragment Length (ZGFL)
15:12 Reserved
ZGFL+15:16 ZoneGroup Fragment (ZGF)
The CDC shall not process received ZoneGroup information until the entire ZoneGroup (i.e., all of the
fragments of the ZoneGroup) is received. Upon receiving the ZoneGroup information, the CDC shall update
that ZoneGroup in ZoneDBActive in that CDC, increment the ZoneGroup generation number, and issue an
FZS command with AAZ operation status cleared to 0h (i.e., Operation Successful) to the DDC.
The AAZ Operation Status is sent through the FZS command and is provided in the FZS buffer, as shown
in Figure 711.
Figure 711: FZS Data for Pull Model AAZ
Bytes Description
00 Operation Type (OTYP): Ah (i.e., AAZ status for a pull model DDC)
03:01 Reserved
07:04 AAZ Operation Status (ZAAOS)
The AAZ Operation Status field is used to encode status information for the pull model AAZ operation, as
shown in Figure 715.
The CDC may enforce access restrictions to the Zoning data structures. In this case, the CDC shall check
if the DDC requesting the AAZ operation is authorized to write the ZoneGroup indicated in Pull Model DDC
Request log page for Pull Model AAZ operation (e.g., if the CDC allows access to a ZoneGroup only to the
DDC that created that ZoneGroup, verify that the ZoneGroup Originator field matches the NQN contained
in the SUBNQN field of the Connect command sent from the CDC to that DDC). If that DDC is not authorized
to access the specified ZoneGroup, then the CDC shall issue an FZS command with AAZ operation status
set to 4h (i.e., ZoneGroup Originator Invalid), and the AAZ operation shall be aborted.
8.3.2.3.8.3.3 Remove Active ZoneGroup (RAZ)
The Remove Active ZoneGroup (RAZ) operation allows a DDC to remove from the CDC an active
ZoneGroup associated with that DDC. For a pull model DDC the RAZ operation is mapped to a Pull Model
DDC Request asynchronous event notification (refer to Figure 151). The CDC responds to that
asynchronous event notification with a Get Log Page command requesting the Pull Model DDC Request
log page (refer to section 5.1.12.3.4), to which the DDC responds with a log page requesting an RAZ
operation for a specified ZoneGroup. The RAZ operation is then completed by the CDC by issuing one FZS
command to provide an operation status to the DDC, as shown in Figure 712.
