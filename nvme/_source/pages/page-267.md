---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 267
headings: []
---

# Source page 267

NVM Express® Base Specification, Revision 2.1

245
Figure 240: Change Namespace Event Data Format (Event Type 06h)
Bytes Description
15:08
Namespace Size (NSZE):  For a create operation, contains the NSZE value from the Identify
Namespace data structure in the Namespace Management command (refer to Figure 369). For a delete
operation that specifies a single namespace this field contains the value from the NSZE field of the
Identify Namespace data  structure (refer to the I/O Command Set specific Identify Namespace data
structure in the applicable I/O Command Set specification ) for the namespace being deleted. For a
delete operation that specifies all namespaces this field is reserved.
23:16 Reserved
31:24
Namespace Capacity (NCAP):  For a creation operation, contains the NCAP value from the Identify
Namespace data structure in the Namespace Management command (refer to Figure 369). For a delete
operation that specifies a single namespace this field contains the value from the NCAP field of the
Identify Namespace data structure (refer to the I/O Command Set specific Identify Namespace data
structure in the applicable I/O Command Set specification ) for the namespace being deleted. For a
delete operation that specifies all namespaces this field is reserved. For I/O Command Sets that do not
define this field, this field is considered reserved.
32
Formatted LBA Size (FLBAS): Refer to the applicable I/O Command Set specification for details. For
I/O Command Sets that do not define this field, this field is considered reserved.
NOTE: This field applies to all User Data Formats . The original name has been retained for historical
continuity.
33
End-to-end Data Protection Type Settings (DPS):  Refer to the applicable I/O Command Set
specification for details. For I/O Command Sets that do not define this field, this field is considered
reserved.
34
Namespace Multi-path I/O and Namespace Sharing Capabilities (NMIC):  For a create operation,
contains the NMIC value from the Namespace Management – Host Software Specified Fields data
structure defined in the applicable I/O Command Set specification. For a delete operation that specifies
a single namespace this field contains the value from the NMIC field of the  I/O Command Set
Independent Identify Namespace data (refer to  Figure 3191) for the namespace being deleted. For a
delete operation that specifies all namespaces this field is reserved.
35 Reserved
39:36
ANA Group Identifier (ANAGRPID):  For a create operation, contains the ANAGRPID value from the
Namespace Management – Host Software Specified Fields data structure defined in the applicable I/O
Command Set specification, if specified, or contains the ANAGRPID value from the  I/O Command Set
Independent Identify Namespace data (refer to Figure 3191) after the namespace was created if an ANA
Group Identifier was not specified in the command. For a delete operation that specifies a single
namespace this field contains the value from the ANAGRPID field of the I/O Command Set Independent
Identify Namespace data (refer to Figure 3191) for the namespace being deleted. For a delete operation
that specifies all namespaces this field is reserved.
If ANA Groups are not supported, then the ANAGRPID field shall be cleared to 0h.
41:40
NVM Set Identifier (NVMSETID):  For a create operation, contains the NVMSETID value from the
Namespace Management – Host Software Specified Fields data structure defined in the applicable I/O
Command Set specification. For a delete operation that specifies a single namespace this field contains
the value from the NVMSETID field of the Identify Namespace data (refer to  Figure 3191) for the
namespace being deleted. For a delete operation that specifies all namespaces this field is reserved.
43:42
Endurance Group Identifier (ENDGID): For a create operation, contains the ENDGID value from the
Namespace Management – Host Software Specified Fields data structure defined in the applicable I/O
Command Set specification. For a delete operation that specifies a single namespace, this field contains
the value from the ENDGID field of the Identify Namespace data structure (refer to Figure 3191) for the
namespace being deleted. For a delete operation that specifies all namespaces this field is reserved.
47:44
Namespace ID (NSID): For a create operation, contains the NSID for the namespace that was created.
For a delete operation, contains the NSID for the namespace being deleted or FFFFFFFFh for a delete
operation specifying all namespaces.
