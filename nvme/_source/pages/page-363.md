---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 363
headings: ["5.1.13.2.16 Endurance Group List (19h)"]
---

# Source page 363

NVM Express® Base Specification, Revision 2.1

341
Figure 323: Domain List
Bytes Description
Domain Attributes Entry List
255:128 Entry 0: This field contains the first Domain Attributes Entry in the list, if present.
383:256 Entry 1: This field contains the second Domain Attributes Entry in the list, if present.
… …
((NUMENT-1)*128+255):
((NUMENT-1)*128+128)
Entry NUMENT-1: This field contains the last Domain Attributes Entry in the list, if
present.

Figure 324: Domain Attributes Entry
Bytes Description
01:00 Domain Identifier (DID): This field indicates the identifier of the Domain accessible by the controller
that is described by this entry.
15:02 Reserved
31:16 Total Domain Capacity  (TDC): This field indicates the total NVM capacity in this Domain. The
value is in bytes.
47:32 Unallocated Domain Capacity  (UDC): This field indicates the unallocated NVM capacity in this
Domain. The value is in bytes.
63:48
Max Endurance Group Domain Capacity  (MEGDC): This field indicates the maximum capacity
of a single Endurance Group in this Domain. If this field is cleared to 0h, t hen the NVM subsystem
does not report a maximum Endurance Group Domain Capacity value.
127:64 Reserved
5.1.13.2.16 Endurance Group List (19h)
An Endurance Group List (refer to  Figure 326) of up to 2,047 Endurance Group Identifiers in increasing
order is returned containing an Endurance Group Identifier greater than or equal to the Endurance Group
Identifier specified by the CNS Specific Identifier field as defined in Figure 325. The list contains Endurance
Group Identifiers of Endurance Groups that are accessible by the controller processing the command. If
the value specified in the Endurance Group Identifier is greater than ENDGIDMAX, then the controller shall
complete the command with a status code of Successful Completion and return an Endurance Group List
containing no Endurance Group Identifiers.
Figure 325: Command Dword 11 - CNS Specific Identifier
Bits Description
15:0
Endurance Group Identifier (ENGGID) : This field specifies the Endurance Group Identifier  (refer to
section 3.2.3) of the first Endurance Group of the ordered list of Endurance Group Identifiers to be
returned.

Figure 326: Endurance Group List
Bytes Description
Header
01:00
Number of Endurance Group Identifiers (N EGIDS): This field contains the number of
Endurance Group Identifiers in the list. There may be up to 2,047 identifiers in the list. If this field
is cleared to 0h, then no Endurance Group Identifiers are in the list.
Endurance Group Identifier List
03:02 Identifier 0: This field contains the first Endurance Group Identifier in the list, if any.
05:04 Identifier 1: This field contains the second Endurance Group Identifier in the list, if any.
… …
(NEGIDS*2+1):
(NEGIDS *2) Identifier NEGIDS-1: This field contains the last Endurance Group Identifier in the list.
