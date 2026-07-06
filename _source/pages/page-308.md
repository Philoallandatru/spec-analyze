---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 308
headings: []
---

# Source page 308

NVM Express® Base Specification, Revision 2.1

286
If an NVM subsystem supports the dynamic controller model, then all entries for that NVM subsystem shall
have the Controller ID field set to FFFFh. For a particular NVM subsystem port and NVMe Transport
address in an NVM subsystem, there shall be no more than one entry with the Controller ID field set to:
• FFFFh if that NVM subsystem supports the dynamic controller model; or
• FFFEh if that NVM subsystem supports the static controller model.
Figure 293: Discovery Log Specific Parameter Field
Bits Description
14:11 Reserved
10
All NVM Subsystem Entries (ALLSUBE): If this bit is set to ‘1’, then the Discovery controller shall
return records for all NVM subsystem ports that are registered without filtering the list of returned
records based upon the requesting HOSTNQN. If this bit is cleared to ‘0’, then this bit has no effect.
09
Port Local Entries Only (PLEO):  If this bit is set to ‘1’, then the Discovery controller shall return
records for only NVM subsystem ports that are presented through the same NVM subsystem port that
received the Get Log Page command. If this bit is cleared to ‘0’, then this bit has no effect.
08
Extended Discovery Log Page Entries (EXTDLPE): If this bit is set to ‘1’, then each record returned
by the Discovery controller may be either a Discovery Log Page Entry or an Extended Discovery Log
Page Entry. If this bit is cleared to ‘0’, then this bit has no effect.

Figure 294: Discovery Log Page Entry Data Structure
Bytes Description
00
Transport Type (TRTYPE): Specifies the NVMe Transport type.
Value Definition
00 Reserved
01 RDMA Transport (refer to the NVMe RDMA Transport specification)
02 Fibre Channel Transport (refer to INCITS 556)
03 TCP Transport (refer to the NVMe TCP Transport specification)
04 to 253 Reserved
254 Intra-host Transport (i.e., loopback) (NOTE: This is a reserved value for use by
host software.)
255 Reserved

01
Address Family (ADRFAM): Specifies the address family.
Value Definition
00 Reserved
01 AF_INET: IPv4 address family. IPv4address format syntax specified in section
3.2.2 of IETF RFC 3986.
02 AF_INET6: IPv6 address family. IPv6address format syntax specified in section
3.2.2 of IETF RFC 3986.
03 AF_IB: InfiniBand address family.
04 Fibre Channel address family.
05 to 253 Reserved
254 Intra-host Transport (i.e., loopback) (NOTE: This is a reserved value for use by
host software.)
255 Reserved
