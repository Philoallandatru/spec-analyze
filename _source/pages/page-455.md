---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 455
headings: []
---

# Source page 455

NVM Express® Base Specification, Revision 2.1

433
Figure 498: Extended Discovery Information Entry
Bytes O/M1 Description
1023:768 M
Transport Specific Address Subtype (TSAS): This field specifies NVMe
Transport specific information about the address as defined in the
Transport Specific Address Subtype (TSAS) field in Figure 294.
1027:1024 M Total Entry Length (TEL) : This field specifies the length in bytes of the
entire extended discovery information entry.
1029:1028 M3
Number of Extended Attributes (NUMEXAT):  This field specifies the
number of extended attributes contained in the extended discovery
information entry.
This field shall be set to a non -zero value (i.e., the extended discovery
information entry shall contain at least one extended attribute) if the Entity
Type (ETYPE) field in the Discovery Information Registration command
data portion’s header is set to 1h  or 3h (i.e., a host or CDC is performing
the register, de-register, or update task). If the ETYPE field is set to 1h or
3h and this field is cleared to 0h, then the controller shall abort the
command with a status code of Invalid Discovery Information.
1031:1030  Reserved
Extended Attribute List
((EXATLEN - 1) + 4) + 1032:
1032 M3
Extended Attribute 0:  This field contains the first extended attribute as
defined in Figure 499 (if present), where EXATLEN is the size specified in
the Extended Attribute Length (EXATLEN) field of the extended attribute.
… O …
TEL - 1:
TEL - (EXATLEN + 4) O
Extended Attribute N:  This field contains the Nth extended attribute as
defined in Figure 499 (if present), where EXATLEN is the size specified in
the Extended Attribute Length (EXATLEN) field of the extended attribute
and TEL is the size specified in the Total Entry Length (TEL) field.
Notes:
1. O/M definition: O = Optional, M = Mandatory.
2. This field is mandatory for NVM subsystem discovery information entries. For host discovery information entries,
this field shall be cleared to 0h.
3. This field is mandatory for host discovery information entries and optional for NVM subsystem discovery
information entries.

Figure 499: Extended Attribute
Bytes Description
01:00
Extended Attribute Type (EXATTYPE): This field specifies the type of extended
attribute for the extended attribute contained in the extended discovery information entry.
Value O/M1 Definition
0h  Reserved
1h HM Host Identifier
2h O Admin label ASCII
3h O Admin label UTF-8
4h to FEFFh  Reserved
FF00h to FFFFh O Vendor Specific
