---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 653
headings: []
---

# Source page 653

NVM Express® Base Specification, Revision 2.1

631
Figure 708: AAZ for Pull Model DDC
CDC DDC
AEN(Pull_DDC_Request)
Get_Log_Page(SQE:Pull_DDC_Request)
Get_Log_Page(Data:AAZ, T_ID, ZoneGroupID)
FZR(SQE:ZDK=T_ID)
FZR(Data:AAZ, ZoneGroup)
FZS(SQE:ZDK=T_ID; Data:AAZ, Status)
FZS()

The format of the operation specific parameters of a Pull Model DDC Request log page requesting an AAZ
is shown in Figure 709.
Figure 709: AAZ Operation Specific Parameters for Pull Model DDC Request Log Page
Bytes Description
03:00 Transaction ID (T_ID)
227:04 ZoneGroup Originator (ZGORIG)
257:228 ZoneGroup Name (ZGNAME)
The Transaction ID field is used to relate the Pull Model DDC Request log page for a pull model AAZ
operation to the subsequent FZR and FZS command(s). The Zoning Data Key in Command Dword 10
(refer to Figure 506) of the FZR command(s) shall be set to the received Transaction ID value. The Zoning
Data Key in Command Dword 10 (refer to Figure 512) of the FZS command shall be set to the received
Transaction ID value.
If the specified ZoneGroup is locked on the CDC, then the AAZ operation shall be aborted by the CDC by
issuing one FZS command with AAZ operation status set to 3h (i.e., Zoning Data Structure Locked) to the
DDC. The Transaction ID value returned by the DDC  in the Pull Model DDC Request log page for a pull
model AAZ operation shall be the same in the FZS command for this AAZ operation.
If the specified ZoneGroup:
• does not exist in ZoneDBActive in that CDC; or
• exists in ZoneDBActive in that CDC and is not locked by another administrative interface,
then the CDC shall lock the specified ZoneGroup in ZoneDBActive and complete the AAZ operation by
issuing one or more FZR commands to request from the DDC the ZoneGroup definition to add/replace,
followed by an FZS command to report status information to t he DDC. The ZoneGroup definition is sent
through one or more FZR commands and is provided in the FZR buffer, as shown in Figure 710. The FZR
completion queue entry sending the last fragment shall have the Last Fragment (LF) bit set to ‘1’ in
Completion Queue Entry Dword 0 (refer to Figure 182).
