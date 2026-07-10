---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 655
headings: []
---

# Source page 655

NVM Express® Base Specification, Revision 2.1

633
Figure 712: RAZ for Pull Model DDC
CDC DDC
AEN(Pull_DDC_Request)
Get_Log_Page(SQE:Pull_DDC_Request)
Get_Log_Page(Data:RAZ, T_ID, ZoneGroupID)
FZS(SQE:ZDK=T_ID; Data:RAZ, Status)
FZS()

The format of the operation specific parameters of a Pull Model DDC Request log page requesting an RAZ
operation is shown in Figure 713.
Figure 713: RAZ Operation Specific Parameters for Pull Model DDC Request Log Page
Bytes Description
03:00 Transaction ID (T_ID)
227:04 ZoneGroup Originator (ZGORIG)
257:228 ZoneGroup Name (ZGNAME)
The RAZ operation is then completed by the CDC by issuing one FZS command to report status information
to the DDC. The Transaction ID field is used to relate the Pull Model DDC Request log page for a pull model
RAZ operation to the subsequent FZS command. The Zoning Data Key in Command Dword 10 (refer to
Figure 512) of the FZS command shall be set to the received Transaction ID value.
The RAZ Operation Status is sent through the FZS command and is provided in the FZS buffer, as shown
in Figure 714.
Figure 714: FZS Data for Pull Model RAZ
Bytes Description
00 Operation Type (OTYP): Ch (i.e., RAZ status for a pull model DDC)
03:01 Reserved
07:04 RAZ Operation Status (RAZOS)
The RAZ Operation Status field is used to encode status information for the pull model RAZ operation, as
shown in Figure 715.
If the ZoneGroup indicated in the Pull Model DDC Request log page for a pull model RAZ operation does
not exist on the CDC, then the CDC shall issue an FZS command with RAZ operation status set to 2h (i.e.,
Zoning Data Structure Not Found), and the RAZ operation shall be aborted.
If the ZoneGroup indicated in the Pull Model DDC Request log page for a pull model RAZ operation is
locked on the CDC (i.e., another administrative interface is modifying that ZoneGroup), then the CDC shall
issue an FZS command with RAZ operation status se t to 3h (i.e., Zoning Data Structure Locked), and the
RAZ operation shall be aborted
If the ZoneGroup indicated in the Pull Model DDC Request log page for a pull model RAZ operation is not
locked on the CDC, then the CDC shall continue the GAZ operation by removing the requested ZoneGroup
