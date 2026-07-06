---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 645
headings: ["8.3.2.3.4.10 {ZoneAlias} Zone Member Type", "8.3.2.3.5 Zone Properties", "8.3.2.3.5.1 Overview"]
---

# Source page 645

NVM Express® Base Specification, Revision 2.1

623
Figure 692: {(IP+Protocol+IP Protocol Port), Role} Zone Member Format
Bytes Description
00 Zone Member Type (ZMTYPE): This field specifies the Zone member type and shall be set to 12h.
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
05
Zone Member IP Protocol (ZMIPPROTO): This field specifies the IP protocol. This field shall be set to
one of the following values:
• 6h: TCP; or
• 11h: UDP.
07:06 Zone Member Transport Service Identifier (ZMTRSVCID): This field specifies the TCP or UDP port.
23:08
Zone Member Transport Address (ZMTRADDR): This field specifies the IP address. An IPv6 address
is encoded in binary as follows:
Bytes Description
15:00 IPv6 Address (IPV6ADDR): This field contains an IPv6 address.
An IPv4 address is encoded in binary as follows:
Bytes Description
03:00 IPv4 Address (IPV4ADDR): This field contains an IPv4 address.
15:04 Reserved

255:24 Reserved
 {ZoneAlias} Zone Member Type
This Zone member type is used to identify a ZoneAlias in a Zone definition. The format of this Zone member
type is shown in Figure 693.
Figure 693: {ZoneAlias} Zone Member Format
Bytes Description
00 Zone Member Type (ZMTYPE): This field specifies the Zone member type and shall be set to EFh.
02:01 Zone Member Length (ZMLEN): This field specifies the length in bytes of this Zone member type and
shall be set to 100h.
31:03 Reserved
155:32 Zone Member ZoneAlias Name (ZMZANAME):  This field specifies an ASCII encoded null -terminated
string, NULL padded as necessary to the 124-byte maximum length.
255:156 Reserved
8.3.2.3.5 Zone Properties
 Overview
Zone properties are represented as type-length-value (TLV) data structures. Unrecognized Zone properties
may be ignored. Figure 694 shows the defined Zone property types.
Figure 694: Zone Property Types
Type Definition
1h Fabric Enforced Zone
