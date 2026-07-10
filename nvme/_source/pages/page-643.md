---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 643
headings: ["8.3.2.3.4.6 {(NQN, PortID), Role} Zone Member Type (Type 5h)", "8.3.2.3.4.7 {(NQN, Protocol, PortID, ADRFAM), Role} Zone Member Type (Type 6h)"]
---

# Source page 643

NVM Express® Base Specification, Revision 2.1

621
Figure 688: {(NQN+IP+Protocol+IP Protocol Port+PortID), Role} Zone Member Format
Bytes Description
23:08
Zone Member Transport Address (ZMTRADDR): This field specifies the IP address. An IPv6 address
is encoded in binary as follows:
Bytes Description
15:00 IPv6 Address (IPV6ADDR): This field contains an IPv6 address.
An IPv4 address is encoded in binary as follows:
Bytes Description
03:00 IPv4 Address (IPV4ADDR): This field contains an IPv4 address.
15:04 Reserved

25:24 Zone Member PortID (ZMPORTID): This field specifies the NVM subsystem PortID.
31:26 Reserved
255:32 Zone Member NQN (ZMNQN):  This field specifies the NQN (represented as a null -terminated string,
NULL padded as necessary to the 224-byte maximum length) of the referenced NVMe-oF entity.
 {(NQN, PortID), Role} Zone Member Type (Type 5h)
This Zone member type identifies a specific PortID used by the NVMe -oF entity identified by the Zone
member’s NQN. The format of this Zone member type is shown in Figure 689.
Figure 689: {(NQN+PortID), Role} Zone Member Format
Bytes Description
00 Zone Member Type (ZMTYPE): This field specifies the Zone member type and shall be set to 5h.
02:01 Zone Member Length (ZMLEN):  This field specifies the length in bytes of this Zone member type and
shall be set to 100h.
03
Zone Member Role (ZMROLE): This field specifies what type of entity the Zone member is. This field
shall be set to one of the following values:
• 1h: host; or
• 2h: NVM subsystem.
05:04 Zone Member PortID (ZMPORTID): This field specifies the NVM subsystem PortID.
31:06 Reserved
255:32 Zone Member NQN (ZMNQN):  This field specifies the NQN (represented as a null -terminated string,
NULL padded as necessary to the 224-byte maximum length) of the referenced NVMe-oF entity.
 {(NQN, Protocol, PortID, ADRFAM), Role} Zone Member Type (Type 6h)
This Zone member type identifies the specific IP protocol (e.g., TCP), PortID, and Address Family used by
the NVMe-oF entity identified by the Zone member’s NQN over that fabric interface. The format of this Zone
member type is shown in Figure 690.
Figure 690: {(NQN+Protocol+PortID+ADRFAM), Role} Zone Member Format
Bytes Description
00 Zone Member Type (ZMTYPE): This field specifies the Zone member type and shall be set to 6h.
02:01 Zone Member Length (ZMLEN):  This field specifies the length in bytes of this Zone member type and
shall be set to 100h.
03
Zone Member Role (ZMROLE): This field specifies what type of entity the Zone member is. This field
shall be set to one of the following values:
• 1h: host; or
• 2h: NVM subsystem.
04
Zone Member Address Family (ZMADRFAM): This field specifies the IP address family. This field shall
be set to one of the following values:
• 1h: IPv4 address family; or
• 2h: IPv6 address family.
