---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 647
headings: ["8.3.2.3.8.2 Push Model DDC Operations", "8.3.2.3.8.2.1 Get Active ZoneGroup (GAZ)"]
---

# Source page 647

NVM Express® Base Specification, Revision 2.1

625
specification or the protocol defined in this specification), the ZoneGroup may become locked to access
from any other interface.
Management of ZoneGroups generally happens through an administrative interface on the CDC. Push
model Direct Discovery controllers (DDCs) may be able to manage their own ZoneGroups (i.e., ZoneGroups
having the DDC NQN as ZoneGroup Originator) as specified in section 8.3.2.3.8.2. Pull model DDCs may
be able to manage their own ZoneGroups (i.e., ZoneGroups having the DDC NQN as ZoneGroup
Originator) as specified in section 8.3.2.3.8.3.
By default, a ZoneGroup should be accessible only to an administrative interface outside the scope of this
specification on the CDC and to the ZoneGroup originator DDC through the protocol defined in this
specification.
If Fabric Zoning is not enabled on the CDC, then that CDC:
• shall abort all Fabric Zoning commands (i.e., any Fabric Zoning command that is issued as part of
a Zoning operation) issued by a push model DDC with a status code of Requested Function
Disabled; and
• shall not react to Pull Model DDC Request asynchronous event notifications issued by a pull model
DDC.
 Push Model DDC Operations
8.3.2.3.8.2.1 Get Active ZoneGroup (GAZ)
The Get Active ZoneGroup (GAZ) operation allows a DDC to retrieve from the CDC an active ZoneGroup
associated with the DDC initiating the GAZ operation. For a push model DDC, the GAZ operation is mapped
to an FZL command to lookup the key of the ZoneGroup to retrieve in ZoneDBActive in that CDC, followed
by one or more FZR commands to retrieve that ZoneGroup from the CDC, as shown in Figure 697.
Figure 697: GAZ for Push Model DDC
CDC DDC
FZL(Data:GAZ, ZoneGroupID)
FZL(CQE:ZDK=ZoneGroup_Key)
FZR(SQE:ZDK=ZoneGroup_Key)
FZR(Data:GAZ, ZoneGroupID)

The identifier of the ZoneGroup to get is provided in the FZL buffer, as shown in Figure 698.
Figure 698: FZL Data for GAZ
Bytes Description
00 Operation Type (OTYP): 1h (i.e., Lookup for Get Active ZoneGroup)
03:01 Reserved
227:04 ZoneGroup Originator (ZGORIG)
257:228 ZoneGroup Name (ZGNAME)
