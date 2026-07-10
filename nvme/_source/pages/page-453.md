---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 453
headings: []
---

# Source page 453

NVM Express® Base Specification, Revision 2.1

431
Figure 497: Discovery Information Management – Data
Bytes O/M1 Description
279:24 M
Entity Identifier (EID): This field specifies an NQN that is used to uniquely identify the
entity performing the registration, de -registration, or update task within the fabric in
UUID-based format. Refer to section 4.7 for the formatting requirements of UUID -
based format NQNs.
535:280 O
Entity Name (ENAME): This field specifies the name associated with the entity as an
ASCII string (refer to section 1.4.2 for ASCII string requirements). The value specified
in this field is determined by the value set in the Entity Type (ETYPE) field.
If the ETYPE field is set to 1h (i.e., a host is performing the register, de -register, or
update task), then this field specifies the name associated with the host.
If the ETYPE field is set to 2h (i.e., a DDC is performing the register, de -register, or
update task), then this field specifies the name associated with the DDC.
If the ETYPE field is set to 3h (i.e., a CDC is performing the register, de -register, or
update task), then this field specifies the name associated with the CDC.
599:536 O
Entity Version (EVER): This field specifies the entity version as an ASCII string (refer
to section 1.4.2 for ASCII string requirements). The value specified in this field is
determined by the value set in the Entity Type (ETYPE) field.
If the ETYPE field is set to 1h (i.e., a host is performing the register, de -register, or
update task), then this field specifies the host entity version (e.g., host operating
system name and version).
If the ETYPE field is set to 2h (i.e., a DDC is performing the register, de -register, or
update task), then this field specifies the Discovery controller entity version (e.g.,
currently active firmware revision of the DDC).
If the ETYPE field is set to 3h (i.e., a CDC is performing the register, de -register, or
update task), then this field may specify either:
a) the host entity version (e.g., host operating system name and version); or
b) the Discovery controller entity version (e.g., currently active firmware revision
of the CDC).
1023:600  Reserved
Discovery Information Entry List
(TEL - 1) + 1024:
1024 M
Discovery Information Entry 0: This field contains the first discovery information
entry as defined in Figure 497 for extended discovery information entries and as
defined in Figure 294 for basic discovery information entries, where TEL is the size
specified in the Total Entry Length (TEL) field of the extended discovery information
entry. For basic discovery information entries, TEL shall be 1,024 bytes.
… O …
TDL - 1:
TDL - TEL  M3
Discovery Information Entry N UMENT-1: This field contains the last discovery
information entry as defined in Figure 498 for extended discovery information entries
(if present) and as defined in Figure 294 for basic discovery information entries (if
present), where TEL is the size specified in the Total Entry Length (TEL) field of the
extended discovery information entry and TDL is the size specified in the Total Data
Length (TDL) field. For basic discovery information entries, TEL shall be 1,024 bytes.
Notes:
1. O/M definition: O = Optional, M = Mandatory.
2. This field is optional for a register, de -register, or update task performed by a DDC. For a register, de -register,
or update task performed by a host or CDC, this field shall be cleared to 0h.
3. This field is mandatory for an update task. For a register or de-register task, this field is optional.
