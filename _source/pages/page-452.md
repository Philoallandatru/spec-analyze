---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 452
headings: []
---

# Source page 452

NVM Express® Base Specification, Revision 2.1

430
Figure 497: Discovery Information Management – Data
Bytes O/M1 Description
17:16 M
Entry Format (ENTFMT): This field specifies the format of the discovery information
entries being registered, de-registered, or updated.
Value Definition
0h Reserved
1h Basic discovery information entry
2h Extended discovery information entry
3h to FFFFh Reserved

19:18 M
Entity Type (ETYPE): This field specifies the type of entity performing the register, de-
register, or update task.
Value Definition
0h Reserved
1h Host
2h Direct Discovery controller
3h Centralized Discovery controller
4h to FFFFh Reserved

20 O2
Port Local (PORTLCL): This field specifies if the NVM subsystem discovery
information entries being registered, de -registered, or updated are only for NVM
subsystem ports that are presented through the same NVM subsystem port on the
DDC that is performing the register, de -register, or update task. If the Entity Type
(ETYPE) field is set to any value other than 2h (i.e., an entity other than a DDC is
performing the register, de-register, or update task) and this field is not cleare d to 0h,
then the controller shall abort the command with a status code of Invalid Field in
Command.
If this field is set to 1h, then the NVM subsystem discovery information entries being
registered, de -registered, or updated are only for NVM subsystem ports that are
presented through the same NVM subsystem port on the DDC that is performing the
register, de-register, or update task.
If this field is cleared to 0h, then the NVM subsystem discovery information entries
being registered, de-registered, or updated may be for NVM subsystem ports that are
presented through any NVM subsystem port on that DDC.
21  Reserved
23:22 M
Entry Key Type (EKTYPE): This field specifies the fields in each discovery information
entry that shall be used to determine if the entry matches an existing registration record
contained in the CDC or DDC. Host discovery information entries shall use an EKTYPE
of 5Fh (i.e., TRADDR Based). NVM subsystem discovery information entries may use
an EKTYPE of either 3Fh (i.e., Port ID Based) or 5Fh (i.e., TRADDR Based). All other
values are reserved.
Bits Description
Port ID
Based
(3Fh)
TRADDR
Based
(5Fh)
15:7 Reserved
6 Transport Address (TRADDR) Not used Used
5 Port ID (PORTID) Used Not used
4 Transport Type (TRTYPE)
Used Used
3 Address Family (ADRFAM)
2 Transport Service Identifier (TRSVCID)
1 Transport Specific Address Subtype (TSAS)
0 NVMe Qualified Name (NQN)
