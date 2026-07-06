---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 361
headings: ["5.1.13.2.9 Allocated Namespace ID list (CNS 10h)", "5.1.13.2.10 Identify Namespace data structure for an Allocated Namespace ID (CNS 11h)", "5.1.13.2.11 Namespace Attached Controller list (CNS 12h)", "5.1.13.2.12 Controller list (CNS 13h)", "5.1.13.2.13 Namespace Granularity List (CNS 16h)", "5.1.13.2.14 UUID List (CNS 17h)"]
---

# Source page 361

NVM Express® Base Specification, Revision 2.1

339
5.1.13.2.9 Allocated Namespace ID list (CNS 10h)
A list of up to 1,024 namespace IDs is returned to the host containing allocated NSIDs in increasing order
that are greater than the value specified in the Namespace Identifier (NSID) field of the Identify command.
The controller shall abort the command with a status code of Invalid Namespace or Format if the NSID field
is set to FFFFFFFEh or FFFFFFFFh. The NSID field may be cleared to 0h to retrieve a Namespace List
including the namespace starting with NSID of 1h. The data structure returned is a Namespace List (refer
to section 4.6.2).
5.1.13.2.10 Identify Namespace data structure for an Allocated Namespace ID (CNS 11h)
The Identify Namespace data structure (refer to the NVM Command Set Specification) is returned to the
host for the specified namespace if the value in the Namespace Identifier (NSID) field is an allocated NSID.
If the value in the NSID field specifies an unallocated NSID , then the controller returns a zero filled data
structure. If the specified namespace is not associated with an I/O Command Set that specifies logical
blocks (e.g., the NVM Command Set ), then the controller shall abort the command with a status code of
Invalid I/O Command Set.
If the value in the NSID field specifies an invalid NSID , then the controller shall abort the command with a
status code of Invalid Namespace or Format. If the NSID field is set to FFFFFFFFh, then the controller shall
abort the command with a status code of Invalid Namespace or Format.
5.1.13.2.11 Namespace Attached Controller list (CNS 12h)
A Controller List (refer to section 4.6.1) of up to 2,047 controller identifiers is returned containing a controller
identifier greater than or equal to the value specified in the Controller Identifier (CDW10.CNTID) field. The
list contains controller identifiers of controllers that have the specified namespace attached . If the NSID
field is set to FFFFFFFFh, then the controller shall abort the command with a status code of Invalid Field in
Command.
5.1.13.2.12 Controller list (CNS 13h)
A Controller List (refer to section  4.6.1) of up to 2 ,047 controller identifiers of I/O controllers is returned
containing controller identifier s greater than or equal to the value specified in the Controller Identifier
(CDW10.CNTID) field.
5.1.13.2.13 Namespace Granularity List (CNS 16h)
If the controller supports reporting of Namespace Granularity  refer to the applicable I/O Command Set
specification for details.
The controller shall abort the command with a status code of Invalid I/O Command Set if the Command Set
Identifier is not associated with an I/O Command Set that supports the Namespace Granularity List.
5.1.13.2.14 UUID List (CNS 17h)
The format of the UUID List is defined in Figure 320. Each UUID List entry is either 0h, the NVMe Invalid
UUID, or a valid UUID. Valid UUIDs are those which are non-zero and are not the NVMe Invalid UUID (refer
to section 8.1.28).
If the UUID List bit is set to ‘1’ in the Controller Attributes (CTRATT) field in the Identify Controller data
structure (refer to Figure 312), then:
• The UUID List shall contain at least one valid UUID (refer to section 8.1.28);
• The UUID 1 field shall contain a non-zero value; and
• A UUID field cleared to 0h indicates the end of the UUID List.
The list may be in any order.
