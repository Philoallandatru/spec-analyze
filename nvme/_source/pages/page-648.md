---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 648
headings: ["8.3.2.3.8.2.2 Add/Replace Active ZoneGroup (AAZ)"]
---

# Source page 648

NVM Express® Base Specification, Revision 2.1

626
The FZL command returns the key of the ZoneGroup to retrieve as a Zoning Data key value in the
Completion Queue. For the FZL command of a GAZ operation:
• if the requested ZoneGroup does not exist on the CDC, then the CDC shall abort the command
with a status code of Zoning Data Structure Not Found; or
• if the requested ZoneGroup is locked on the CDC (i.e., another administrative interface is modifying
that ZoneGroup), then the CDC shall abort the command with a status code of Zoning Data
Structure Locked.
The Zoning Data key value returned by the FZL command is used in Command Dword 10 of an FZR
command (refer to Figure 506) to retrieve that ZoneGroup or a fragment of that ZoneGroup. The ZoneGroup
definition is retrieved through one or more subsequent FZR commands and is returned in the FZR buffer,
as shown in Figure 699. The FZR completion queue entry sending the last fragment shall have the Last
Fragment (LF) bit set to ‘1’ in Completion Queue Entry Dword 0 (refer to Figure 510).
Figure 699: FZR Data for GAZ
Bytes Description
00 Operation Type (OTYP): 2h (i.e., Get Active ZoneGroup for a push model DDC)
03:01 Reserved
07:04 ZoneGroup Generation Number (ZGGN)
11:08 ZoneGroup Fragment Length (ZGFL)
15:12 Reserved
ZGFL+15:16 ZoneGroup Fragment (ZGF)
For the FZR command of a GAZ response operation:
• if the ZoneGroup identified by the specified ZoneGroup key does not exist on the CDC, then the
CDC shall abort the command with a status code of Zoning Data Structure Not Found; or
• if the ZoneGroup identified by the specified ZoneGroup key is locked on the CDC (i.e., another
administrative interface is modifying that ZoneGroup), then the CDC shall abort the command with
a status code of Zoning Data Structure Locked.
When a ZoneGroup is transferred in multiple fragments, the receiver shall verify that the ZoneGroup
generation number stays constant across all FZR commands. If the ZoneGroup generation number
changes, then the GAZ operation shall be aborted. The DDC shall  not process received ZoneGroup
information until the full ZoneGroup (i.e., all of the fragments of the ZoneGroup) is received.
The CDC may enforce access restrictions to the Zoning data structures. In this case, the CDC shall check
if the DDC issuing the FZL command or FZR command is authorized to read the ZoneGroup specified in
the FZL data (e.g., if the CDC allows access to a ZoneGroup only to the DDC that created that ZoneGroup,
verify that the ZoneGroup Originator field matches the NQN contained in the HOSTNQN field of the Connect
command sent from the DDC to that CDC). If that DDC is not authorized to access the specified ZoneGroup,
then the CDC shall abort the FZL command and the FZR command with a status code of ZoneGroup
Originator Invalid.
8.3.2.3.8.2.2 Add/Replace Active ZoneGroup (AAZ)
The Add/Replace Active ZoneGroup (AAZ) operation allows a DDC to add or replace in the CDC an active
ZoneGroup associated with the DDC initiating the AAZ operation. For a push model DDC, the AAZ
operation is mapped to an FZL command to lookup the key of th e ZoneGroup to add or replace in
ZoneDBActive in that CDC, followed by one or more FZS commands to provide the CDC with the
ZoneGroup to add or replace, as shown in Figure 700.
