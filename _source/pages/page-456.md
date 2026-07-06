---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 456
headings: []
---

# Source page 456

NVM Express® Base Specification, Revision 2.1

434
Figure 499: Extended Attribute
Bytes Description
03:02
Extended Attribute Length (EXATLEN): This field specifies the length of the Extended
Attribute Value (EXATVAL) field for the extended attribute contained in the extended
discovery information entry. The length specified in this field shall be a non -zero value
that is a multiple of four, and is either a fixed length or within a variable range based upon
the value set in the Extended Attribute Type (EXATTYPE) field. If the length specified in
this field is not a multiple of four, then the controller shall abort the command with a status
code of Invalid Field in Command.
Extended Attribute Type Length
Host Identifier 16 bytes
Admin label ASCII 4 to 256 bytes Admin label UTF-8

(EXATLEN - 1) + 4:04
Extended Attribute Value (EXATVAL): This field specifies the value for the extended
attribute contained in the extended discovery information entry. The value specified in this
field is based upon the value set in the Extended Attribute Type (EXATTYPE) field.
Unused bytes, if any, shall be cleared to 0h.
Extended
Attribute Type Definition
Host Identifier  This value specifies the Host Identifier of the host, as defined in
section 5.1.25.1.28.
Admin label
ASCII
This value specifies the admin label of the host or NVM subsystem
encoded in ASCII. The admin label is used to identify a host or an
NVM subsystem.
Admin label
UTF-8
This value specifies the admin label of the host or NVM subsystem
encoded in UTF-8. The admin label is used to identify a host or an
NVM subsystem.

Notes:
1. O/M definition: O = Optional, M = Mandatory, HM = Mandatory for hosts and prohibited for NVM subsystems.

Figure 500: Extended Discovery Information Entry – Applicable Fields
Field Name Extended Discovery Information Entry Type:
Host NVM Subsystem
Transport Type (TRTYPE) Used Used
Address Family (ADRFAM) Used Used
Subsystem Type (SUBTYPE) Ignored1 Used
Transport Requirements (TREQ) Ignored1 Used
Port ID (PORTID) Ignored1 Used
Controller ID (CNTLID) Ignored1 Used
Admin Max SQ Size (ASQSZ) Ignored1 Used
Transport Service Identifier (TRSVCID) Ignored1 Used
NVMe Qualified Name (NQN) Used Used
Transport Address (TRADDR) Used Used
Transport Specific Address Subtype (TSAS) Used Used
Total Entry Length (TEL) Used Used
Number of Extended Attributes (NUMEXAT) Used Used
Extended Attribute 0 (EXAT0)…Extended Attribute N (EXATN) Used Used
Notes:
1. This field shall be cleared to 0h.
