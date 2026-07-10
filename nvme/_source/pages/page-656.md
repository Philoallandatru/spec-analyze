---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 656
headings: ["8.3.2.3.8.3.4 Pull Model DDC Zoning Operations Status Values", "8.3.2.3.8.3.5 Pull Model DDC Discovery Log Page Request"]
---

# Source page 656

NVM Express® Base Specification, Revision 2.1

634
from ZoneDBActive in that CDC and by issuing one subsequent FZS command with RAZ operation status
cleared to 0h (i.e., Operation Successful).
The CDC may enforce access restrictions to the Zoning data structures. In this case, the CDC shall check
if the DDC requesting the RAZ operation is authorized to remove the ZoneGroup indicated in the Pull Model
DDC Request log page for Pull Model RAZ operation (e.g., if the CDC allows access to a ZoneGroup only
to the DDC that created that ZoneGroup, verify that the ZoneGroup Originator field matches the NQN
contained in the SUBNQN field of the Connect command sent from the CDC to that DDC). If that DDC is
not authorized to access the specified ZoneGroup, then the CDC shall issue an FZS command with RAZ
operation status set to 4h (i.e., ZoneGroup Originator Invalid), and the RAZ operation shall be aborted.
8.3.2.3.8.3.4 Pull Model DDC Zoning Operations Status Values
The Pull Model DDC Zoning operations status values are listed in Figure 715.
Figure 715: Pull Model DDC Zoning Operations Status Values
Value Definition
0h Operation Successful
1h Operation in Progress
2h Zoning Data Structure Not Found
3h Zoning Data Structure Locked
4h ZoneGroup Originator Invalid
5h ZoneGroup Changed
All others Reserved
8.3.2.3.8.3.5 Pull Model DDC Discovery Log Page Request
A Pull Model DDC retrieves a discovery log page from the CDC through a Pull Model DDC Request
asynchronous event notification (refer to Figure 151). The CDC responds to that asynchronous event
notification with a Get Log Page command requesting the Pull Model DDC Request log page (refer to
section 5.1.12.3.4), to which the DDC responds with a log page requesting a Discovery Log Page Request
operation. The Discovery Log Page Request operation is then completed by the CDC by issuing a SDLP
command (refer to section 5.1.20) to provide the requested discovery log page to the DDC, as shown in
Figure 716.
Figure 716: Log Page Request Operation
CDC DDC
AEN(Pull_DDC_Request)
Get_Log_Page(SQE:Pull_DDC_Request)
Get_Log_Page(Data:Discovery_Log_Page_Request)
SDLP(Data:Requested_Discovery_Log_Page)
SDLP()

The format of the operation specific parameters of a Pull Model DDC Request log page requesting a Log
Page Request operation is shown in Figure 717.
