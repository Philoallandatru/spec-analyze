---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 652
headings: ["8.3.2.3.8.3.2 Add/Replace Active ZoneGroup (AAZ)"]
---

# Source page 652

NVM Express® Base Specification, Revision 2.1

630
Figure 707: FZS Data for Pull Model GAZ
Bytes Description
03:01 Reserved
07:04 GAZ Operation Status (GAZOS)
11:08 ZoneGroup Fragment Length (ZGFL)
15:12 Reserved
ZGFL+15:16 ZoneGroup Fragment (ZGF)
The GAZ Operation Status field is used to encode status information for the pull model GAZ operation, as
shown in Figure 715.
When a ZoneGroup is transferred in multiple fragments, if the CDC detects the ZoneGroup definition has
changed while sending the fragments, then the CDC shall issue an FZS command with a zero length
ZoneGroup fragment and GAZ operation status set to 5h (i. e., ZoneGroup Changed), and the GAZ
operation shall be aborted.
If the ZoneGroup indicated in the Pull Model DDC Request log page for a pull model GAZ operation does
not exist on the CDC, then the CDC shall issue an FZS command with a zero length ZoneGroup fragment
and GAZ operation status set to 2h (i.e., Zoning Data Structure Not Found), and the GAZ operation shall
be aborted.
If the ZoneGroup indicated in the Pull Model DDC Request log page for a pull model GAZ operation is
locked on the CDC (i.e., another administrative interface outside the scope of this specification is modifying
that ZoneGroup), then the CDC shall issue an FZS command with a zero length ZoneGroup fragment and
GAZ operation status set to 3h (i.e., Zoning Data Structure Locked), and the GAZ operation shall be
aborted.
If the ZoneGroup indicated in the Pull Model DDC Request log page for a pull model GAZ operation is not
locked on the CDC, then the CDC shall continue the GAZ operation by issuing one or more subsequent
FZS commands containing the fragments of the requested ZoneGroup. The last fragment shall specify GAZ
operation status cleared to 0h (i.e., Operation Successful), and the other fragments shall specify GAZ
operation status set to 1h (i.e., Operation in Progress).
The DDC shall not process received ZoneGroup information until the entire ZoneGroup (i.e., all of the
fragments of the ZoneGroup) is received.
The CDC may enforce access restrictions to the Zoning data structures. In this case, the CDC shall check
if the DDC requesting the GAZ operation is authorized to read the ZoneGroup indicated in Pull Model DDC
Request log page for Pull Model GAZ operation (e.g., if the CDC allows access to a ZoneGroup only to the
DDC that created that ZoneGroup, verify that the ZoneGroup Originator field matches the NQN contained
in the SUBNQN field of the Connect command sent from the CDC to that DDC). If that DDC is not authorized
to access the specified ZoneGroup, then the CDC shall issue an FZS command with a zero length
ZoneGroup fragment and GAZ operation status set to 4h (i.e., ZoneGroup Originator Invalid), and the GAZ
operation shall be aborted.
8.3.2.3.8.3.2 Add/Replace Active ZoneGroup (AAZ)
The Add/Replace Active ZoneGroup (AAZ) operation allows a DDC to add or replace in the CDC an active
ZoneGroup associated with that DDC. For a pull model DDC the AAZ operation is mapped to a Pull Model
DDC Request asynchronous event notification (refer to Figure 151). The CDC responds to that
asynchronous event notification with a Get Log Page command requesting the Pull Model DDC Request
log page (refer to section 5.1.12.3.4), to which the DDC responds with a log page requesting an AAZ
operation for a specified ZoneGroup. The AAZ operation is then completed by the CDC by issuing one or
more FZR commands to retrieve that ZoneGroup from the DDC, and one FZS command to provide a n
operation status to the DDC, as shown in Figure 708.
