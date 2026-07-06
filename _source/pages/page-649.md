---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 649
headings: []
---

# Source page 649

NVM Express® Base Specification, Revision 2.1

627
Figure 700: AAZ for Push Model DDC
CDC DDC
FZL(Data:AAZ, ZoneGroupID)
FZL(CQE:ZDK=ZoneGroup_Key)
FZS(SQE:ZDK=ZoneGroup_Key, Data:AAZ, ZoneGroup)
FZS()

The identifier of the ZoneGroup to add or replace is provided in the FZL buffer, as shown in Figure 701.
Figure 701: FZL Data for AAZ
Bytes Description
00 Operation Type (OTYP): 3h (i.e., Lookup for Add/Replace Active ZoneGroup)
03:01 Reserved
227:04 ZoneGroup Originator (ZGORIG)
257:228 ZoneGroup Name (ZGNAME)
The FZL command returns the key of the ZoneGroup to add or replace as a Zoning Data key value in the
Completion Queue. For the FZL command of an AAZ operation:
• if the ZoneGroup that matches the specified FZL data exists in ZoneDBActive in that CDC and is
locked by another administrative interface, then the CDC shall abort the command with a status
code of Zoning Data Structure Locked;
• if the ZoneGroup that matches the specified FZL data exists in ZoneDBActive in that CDC and is
not locked by another administrative interface, then that ZoneGroup should be locked by the
submitting DDC and its key shall be returned as the Zoning Data key in the Completion Queue; or
• if the ZoneGroup that matches the specified FZL data does not exist in ZoneDBActive in that CDC,
then an empty ZoneGroup having the provided identifier shall be created in ZoneDBActive in that
CDC, that ZoneGroup should be locked by the submitting DDC, and  its key shall be returned as
the Zoning Data key in the Completion Queue.
Successful completion of the FZL command for the AAZ operation results in the identified ZoneGroup on
the CDC being locked by the DDC performing the operation.
The ZoneGroup to add or replace is provided in the FZS buffer of subsequent FZS commands with the
appropriate Zoning Data key in Command Dword 10 (refer to Figure 512) and is transferred in one or more
fragments, as needed, as shown in Figure 702. The FZS command sending the last fragment shall have
the Last Fragment (LF) bit set to ‘1’ in Command Dword 12 (refer to Figure 514).
Figure 702: FZS Data for AAZ
Bytes Description
00 Operation Type (OTYP): 4h (i.e., Add/Replace Active ZoneGroup for a push model DDC)
03:01 Reserved
NUMD*4+03:04 ZoneGroup Fragment (ZGF)
For the FZS command of an AAZ request operation:
