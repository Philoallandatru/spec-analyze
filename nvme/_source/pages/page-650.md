---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 650
headings: ["8.3.2.3.8.2.3 Remove Active ZoneGroup (RAZ)"]
---

# Source page 650

NVM Express® Base Specification, Revision 2.1

628
• if the ZoneGroup identified by the specified ZoneGroup key does not exist on the CDC, then the
CDC shall abort the command with a status code of Zoning Data Structure Not Found; or
• if the ZoneGroup identified by the specified ZoneGroup key is locked by another entity on the CDC
(i.e., another administrative interface is modifying that ZoneGroup), then the CDC shall abort the
command with a status code of Zoning Data Structure Locked.
The CDC shall update a ZoneGroup and increment its generation number only upon successful reception
of the full ZoneGroup (i.e., all of the fragments of the ZoneGroup). Successful receipt of the full ZoneGroup
for the AAZ operation shall unlock the identif ied ZoneGroup on the CDC. If the full ZoneGroup is not
received within 30 seconds from the establishment of the lock (during processing of the related FZL
command), then all the received data shall be discarded and the lock shall be released (i.e., ZoneDBActive
in that CDC is not changed).
The CDC may enforce access restrictions to the Zoning data structures. In this case, the CDC shall check
if the DDC issuing the FZL command or FZS command is authorized to write the ZoneGroup specified in
the FZL data (e.g., if the CDC allows access to a ZoneGroup only to the DDC that created that ZoneGroup,
verify that the ZoneGroup Originator field matches the NQN contained in the HOSTNQN field of the Connect
command sent from the DDC to that CDC). If that DDC is not authorized to access the specified ZoneGroup,
then the CDC shall abort the FZL command and the FZS command with a status code of ZoneGroup
Originator Invalid.
8.3.2.3.8.2.3 Remove Active ZoneGroup (RAZ)
The Remove Active ZoneGroup (RAZ) operation allows a DDC to remove from the CDC an active
ZoneGroup associated with the DDC initiating the RAZ operation. For a push model DDC, the RAZ
operation is mapped to an FZL command to provide to the CDC with the ide ntifier of the ZoneGroup to
remove, as shown in Figure 703.
Figure 703: RAZ for Push Model DDC
CDC DDC
FZL(Data:RAZ, ZoneGroupID)
FZL()

The identifier of the ZoneGroup to remove is provided in the FZL buffer, as shown in Figure 704.
Figure 704: FZL Data for RAZ
Bytes Description
00 Operation Type (OTYP): 5h (i.e., Lookup for Remove Active ZoneGroup)
03:01 Reserved
227:04 ZoneGroup Originator (ZGORIG)
257:228 ZoneGroup Name (ZGNAME)
For the FZS command of an RAZ request operation:
• if the requested ZoneGroup does not exist on the CDC, then the CDC shall abort the command
with a status code of Zoning Data Structure Not Found; or
• if the requested ZoneGroup is locked on the CDC (i.e., another administrative interface is modifying
that ZoneGroup), then the CDC shall abort the command with a status code of Zoning Data
Structure Locked.
