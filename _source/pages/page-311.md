---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 311
headings: []
---

# Source page 311

NVM Express® Base Specification, Revision 2.1

289
Figure 295: Extended Discovery Log Page Entry
Bytes Description
Header
00 Transport Type (TRTYPE): This field indicates the transport type as defined in the
Transport Type (TRTYPE) field in Figure 294.
01 Address Family (ADRFAM): This field indicates the address family as defined in
the Address Family (ADRFAM) field in Figure 294.
02
Subsystem Type (SUBTYPE): This field indicates the type of the NVM subsystem
that is indicated in this entry as defined in the Subsystem Type (SUBTYPE) field in
Figure 294.
03 Transport Requirements (TREQ): This field indicates requirements for the NVMe
Transport as defined in the Transport Requirements (TREQ) field in Figure 294.
05:04 Port ID (PORTID): This field indicates a particular NVM subsystem port as defined
in the Port ID (PORTID) field in Figure 294.
07:06 Controller ID (CNTLID):  This field indicates the controller ID as defined in the
Controller ID (CNTLID) field in Figure 294.
09:08
Admin Max SQ Size (ASQSZ): This field indicates the maximum size of an Admin
Submission Queue as defined in the Admin Max SQ Size (ASQSZ) field in Figure
294.
11:10 Entry Flags (EFLAGS):  This field indicates additional information related to the
current entry as defined in the Entry Flags (EFLAGS) field in Figure 294.
31:12 Reserved
63:32
Transport Service Identifier (TRSVCID): This field indicates the NVMe Transport
service identifier as an ASCII string as defined in the Transport Service Identifier
(TRSVCID) field in Figure 294.
255:64 Reserved
511:256
NVM Subsystem Qualified Name (NQN ): This field indicates the NVMe Qualified
Name (NQN) that uniquely identifies the NVM subsystem as defined in the NVM
Subsystem Qualified Name (SUBNQN) field in Figure 294.
767:512
Transport Address (TRADDR):  This field indicates the address of the NVM
subsystem that may be used for a Connect command as an ASCII string as defined
in the Transport Address (TRADDR) field in Figure 294.
1023:768
Transport Specific Address Subtype (TSAS): This field indicates NVMe Transport
specific information about the address as defined in the Transport Specific Address
Subtype (TSAS) field in Figure 294.
1027:1024 Total Entry Length (TEL):  This field indicates the length in bytes of the entire
Extended Discovery Log Page Entry.
1029:1028 Number of Extended Attributes (NUMEXAT):  This field indicates the number of
extended attributes contained in the Extended Discovery Log Page Entry.
1031:1030 Reserved
Extended Attribute List
((EXATLEN - 1) + 4) + 1032:
1032
Extended Attribute 0: This field contains the first extended attribute as defined in
Figure 499 (if present), where EXATLEN is the size specified in the Extended
Attribute Length (EXATLEN) field of the extended attribute.
… …
TEL - 1:
TEL - (EXATLEN + 4)
Extended Attribute N: This field contains the Nth extended attribute as defined in
Figure 499 (if present), where EXATLEN is the size specified in the Extended
Attribute Length (EXATLEN) field of the extended attribute and TEL is the size
specified in the Total Entry Length (TEL) field.
