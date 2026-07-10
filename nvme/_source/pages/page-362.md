---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 362
headings: ["5.1.13.2.15 Domain List (CNS 18h)"]
---

# Source page 362

NVM Express® Base Specification, Revision 2.1

340
Figure 320: UUID List
Bytes Description
31:00 Reserved
63:32 UUID 1 (UUID1): This field contains the first UUID List Entry in the list.
95:64 UUID 2  (UUID2): This field contains the second UUID List Entry in the list, if present,
otherwise cleared to 0h.
… …
4063:4032 UUID 126  (UUID126): This field contains the last non -zero UUID List Entry in the list, if
present, otherwise cleared to 0h.
4095:4064 UUID 127 (UUID127): This field shall be cleared to 0h.
The format of a UUID List Entry is defined in Figure 321.
Figure 321: UUID List Entry
Bytes Description
00
UUID Lists Entry Header (ULEH):
Bits Description
7:2 Reserved
1:0
Identifier Association (IDASSOC): This field indicates whether the UUID is associated
with a vendor.

Value Definition
00b No association reported.
01b The UUID is associated with the vendor reported in the PCI Vendor ID field
of the Identify Controller data structure (refer to Figure 312).
10b The UUID is associated with the vendor reported in the PCI Subsystem
Vendor ID field of the Identify Controller data structure.
11b Reserved

15:01 Reserved
31:16 Universally Unique Identifier ( UUID): This field contains a 128 -bit Universally Unique Identifier
(UUID) as specified in RFC 9562. Refer to section 4.5.6.
5.1.13.2.15 Domain List (CNS 18h)
Figure 323 defines a Domain List. The data structure is an ordered list by Domain Identifier, starting with
the first Domain Identifier that is greater than or equal to the Domain Identifier specified by the CNS Specific
Identifier field as defined  in Figure 322 and is accessible by the controller processing the command. The
Domain List describes the attributes for each Domain in the list based on the Domain Attributes Entry in
Figure 324.
Figure 322: Command Dword 11 - CNS Specific Identifier
Bits Description
15:0 Domain Identifier (DID): This field specifies the Domain Identifier  (refer to section 3.2.5.3) of the first
Domain of the ordered list of Domain Attribute Entry data structures to be returned.

Figure 323: Domain List
Bytes Description
Header
00
Number of Entries (NUMENT): This field indicates the number of Domain Attributes
Entries in the list. There are up to 31 entries in the list. A value of 0h indicates that
there are no entries in the list.
127:01 Reserved
