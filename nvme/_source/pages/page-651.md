---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 651
headings: ["8.3.2.3.8.3 Pull Model DDC Operations", "8.3.2.3.8.3.1 Get Active ZoneGroup (GAZ)"]
---

# Source page 651

NVM Express® Base Specification, Revision 2.1

629
The CDC may enforce access restrictions to the Zoning data structures. In this case, the CDC shall check
if the DDC issuing the FZL command is authorized to remove the ZoneGroup specified in the FZL data
(e.g., if the CDC allows access to a ZoneGroup only to the DDC that created that ZoneGroup, verify that
the ZoneGroup Originator field matches the NQN contained in the HOSTNQN field of the Connect
command sent from the DDC to that CDC). If that DDC is not authorized to access the specified ZoneGroup,
then the CDC shall abort the FZL command with a status code of ZoneGroup Originator Invalid.
 Pull Model DDC Operations
8.3.2.3.8.3.1 Get Active ZoneGroup (GAZ)
The Get Active ZoneGroup (GAZ) operation allows a DDC to retrieve from the CDC an active ZoneGroup
associated with that DDC. For a pull model DDC the GAZ operation is mapped to a Pull Model DDC Request
asynchronous event notification (refer to Figure 151). The CDC responds to that asynchronous event
notification with a Get Log Page command requesting the Pull Model DDC Request log page (refer to
section 5.1.12.3.4), to which the DDC responds with a log page requesting a GAZ operation for a specified
ZoneGroup. The GAZ operation is then completed by the CDC by issuing one or more FZS commands to
send that ZoneGroup and the operation status to the DDC, as shown in Figure 705.
Figure 705: GAZ for Pull Model DDC
CDC DDC
AEN(Pull_DDC_Request)
Get_Log_Page(SQE:Pull_DDC_Request)
Get_Log_Page(Data:GAZ, T_ID, ZoneGroupID)
FZS(SQE:ZDK=T_ID; Data:GAZ, ZoneGroup, Status)
FZS()

The format of the operation specific parameters of a Pull Model DDC Request log page requesting a GAZ
operation is shown in Figure 706.
Figure 706: GAZ Operation Specific Parameters for Pull Model DDC Request Log Page
Bytes Description
03:00 Transaction ID (T_ID)
227:04 ZoneGroup Originator (ZGORIG)
257:228 ZoneGroup Name (ZGNAME)
The Transaction ID field is used to relate the Pull Model DDC Request log page for a pull model GAZ to the
subsequent FZS command(s). The Zoning Data Key in Command Dword 10 (refer to Figure 512) in the
FZS command(s) shall be set to the received Transaction ID value. The ZoneGroup definition is sent
through one or more subsequent FZS commands and is provided in the FZS buffer, as shown in Figure
707. The FZS command sending the last fragment shall have the Last Fragment (LF) bit set to ‘1’ in
Command Dword 12 (refer to Figure 514).
Figure 707: FZS Data for Pull Model GAZ
Bytes Description
00 Operation Type (OTYP): 7h (i.e., Get Active ZoneGroup for a pull model DDC)
