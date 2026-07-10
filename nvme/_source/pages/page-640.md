---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 640
headings: ["8.3.2.3.4 Zone Members", "8.3.2.3.4.1 Overview", "8.3.2.3.4.2 {NQN, Role} Zone Member Type (Type 1h)"]
---

# Source page 640

NVM Express® Base Specification, Revision 2.1

618
8.3.2.3.4 Zone Members
 Overview
Zone members are represented as type-length-value (TLV) data structures. Figure 684 shows the defined
Zone member types. If a Zone member type does not include a particular element in the pairing for that
Zone member type, then the element values of that type shall not be used for enforcement of Zoning
restrictions for that Zone (e.g., Zone member type 1h does not include IP addresses in the enforcement of
that Zone member type).
Figure 684: Zone Member Types
Type Definition Reference
1h The pair {NQN, Role} 8.3.2.3.4.2
2h The pair {(NQN, IP, Protocol), Role} 8.3.2.3.4.3
3h The pair {(NQN, IP, Protocol, IP Protocol Port), Role} 8.3.2.3.4.4
4h The pair {(NQN, IP, Protocol, IP Protocol Port, PortID), Role} 8.3.2.3.4.5
5h The pair {(NQN, PortID), Role} 8.3.2.3.4.6
6h The pair {(NQN, Protocol, PortID, ADRFAM), Role} 8.3.2.3.4.7
11h The pair {(IP, Protocol), Role} 8.3.2.3.4.8
12h The pair {(IP, Protocol, IP Protocol Port), Role} 8.3.2.3.4.9
EFh A ZoneAlias name 8.3.2.3.4.10
F0h to FEh Vendor specific
All others Reserved
The members of a Zone may communicate with each other using the following rules:
• hosts may communicate with NVM subsystems;
• NVM subsystems may communicate with hosts;
• hosts shall not communicate with hosts; and
• NVM subsystems shall not communicate with NVM subsystems.
A Zone of a ZoneGroup belonging to the ZoneDBConfig may use all defined Zone member types. When a
ZoneGroup belonging to the ZoneDBConfig is activated and becomes part of the ZoneDBActive, all
ZoneAlias name Zone members are resolved in the group of NVMe e ntities referenced by the ZoneAlias
name.
 {NQN, Role} Zone Member Type (Type 1h)
This Zone member type identifies all fabric interfaces, all IP protocols (e.g., TCP or UDP), and all IP protocol
ports (e.g., TCP port 4420) that may be used by the NVMe-oF entity identified by the Zone member’s NQN.
The format of this Zone member type is shown in Figure 685.
Figure 685: {NQN, Role} Zone Member Format
Bytes Description
00 Zone Member Type (ZMTYPE): This field specifies the Zone member type and shall be set to 1h.
02:01 Zone Member Length (ZMLEN ): This field specifies the length in bytes of this Zone member type and
shall be set to 100h.
03
Zone Member Role (ZMROLE): This field specifies what type of entity the Zone member is. This field
shall be set to one of the following values:
• 1h: host; or
• 2h: NVM subsystem.
31:04 Reserved
255:32 Zone Member NQN (ZMNQN):  This field specifies the NQN (represented as a null -terminated string,
NULL padded as necessary to the 224-byte maximum length) of the referenced NVMe-oF entity.
