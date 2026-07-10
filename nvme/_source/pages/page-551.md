---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 551
headings: ["8.1.15.1 Namespace Management Examples"]
---

# Source page 551

NVM Express® Base Specification, Revision 2.1

529
Set Identifier field in a create operation. If the NVM Set Identifier field is cleared to 0h in a create operation
for a namespace that contains formatted storage, then the controller shall choose the NVM Set from which
to allocate capacity to create that namespace.
If the NVM Set Identifier field and the Endurance Group Identifier field are both cleared to 0h in a create
operation for a namespace that contains formatted storage, then the controller shall choose the Endurance
Group and the NVM Set from which to allocate capacity to create that namespace.
If the NVM Set Identifier field is cleared to 0h and the Endurance Group Identifier field is set to a non -zero
value in a create operation  for a namespace that contains formatted storage , then the controller shall
choose the NVM Set in the specified Endurance Group from which to allocate  capacity to create  that
namespace.
If the NVM Set Identifier field is set to a non-zero value and the Endurance Group Identifier field is cleared
to 0h in a create operation for a namespace that contains formatted storage, then the controller shall abort
the command with a status code of Invalid Field in Command.
If the NVM Set Identifier field and the Endurance Group Identifier field are both set to non -zero values in a
create operation for a namespace that contains formatted storage and the specified NVM Set exists in the
specified Endurance Group, then the controller shall allocate capacity for that created namespace from the
specified NVM Set.
If the NVM Set Identifier field and the Endurance Group Identifier field are both set to non -zero values in a
create operation for a namespace that contains formatted storage and the specified NVM Set does not exist
in the specified Endurance Group, then the controller shall abort the command with a status code of Invalid
Field in Command.
For each namespace  that contains formatted storage , the NVM capacity used for that namespace is
reported in the Identify Namespace data structure (refer to the applicable I/O Command Set specification).
The controller may allocate NVM capacity in units such that the requested size for a namespace  that
contains formatted storage may be rounded up to the next unit boundary. The units in which NVM capacity
is allocated are reported in the Namespace Granularity List (refer to the NVM Command Set Specification),
if supported. For example, when using the NVM Command Set, if host software requests a namespace of
32 logical blocks with a logical block size of 4  KiB for a total size of 128  KiB and the allocation unit for the
implementation is 1 MiB, then the NVM capacity consumed may be rounded up to 1 MiB. The NVM capacity
fields may not correspond to the logical block size multiplied by the total number of logical blocks.
The method of allocating ANA Group identifiers is outside the scope of this specification. If the ANA Group
Identifier (refer to Figure 319 and th e Identify Namespace data structure in the NVM Command Set
Specification) is cleared to 0h, then the controller shall determine the ANAGRPID that is assigned to that
namespace.
 Namespace Management Examples
The following is an example of how a host creates a namespace:
1. the h ost requests the Identify Namespace data structure that specifies common namespace
capabilities (i.e., using an Identify command with the NSID field set to FFFFFFFFh and the CNS
field cleared to 0h);
2. if the controller supports reporting of I/O Command Set specific Namespace Management content
(refer to the Namespace Management section in the applicable I/O Command Set specification),
host software optionally requests that information (e.g., Namespace Granularity).
3. the host determines available resources (e.g., capacity for a namespace that contains formatted
storage) (refer to section 3.8);
4. the host creates the data structure defined in Figure 369 (e.g., taking into account the common
namespace capabilities, available capacity);
5. the host issues the Namespace Management command specifying the Create operation and the
data structure. On successful completion of the command, the Namespace Identifier of the new
namespace is returned in Dword 0 of the completion queue entry. At this point, the new namespace
is not attached to any controller; and
