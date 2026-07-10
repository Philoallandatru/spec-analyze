---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 355
headings: ["5.1.13.2.5 I/O Command Set specific Identify Namespace data structure (CNS 05h)"]
---

# Source page 355

NVM Express® Base Specification, Revision 2.1

333
field as defined in Figure 316 and are accessible by the controller processing the command. The NVM Set
List describes the attributes for each NVM Set in the list based on the NVM Set Attributes Entry in Figure
317.
Figure 316: Command Dword 11 - CNS Specific Identifier
Bits Description
15:0 NVM Set Identifier (NVMSETID):  This field specifies the NVM Set Identifier  (refer to section 3.2.2) of
the first NVM Set of the ordered list of NVM Set Attribute Entry data structures to be returned.
The NVM Set List shall not contain an entry cleared to 0h.
Figure 317: NVM Set List
Bytes Description
Header
00
Number of Entries (NUMENT) : This field indicates the number of NVM Set
Attributes Entries in the list. There are up to 31 entries in the list. A value of 0 h
indicates that there are no entries in the list.
127:01 Reserved
NVM Set Attributes Entry List
255:128 Entry 0: This field contains the first NVM Set Attributes Entry in the list, if present.
383:256 Entry 1: This field contains the second NVM Set Attributes Entry in the list, if present.
… …
((NUMENT-1)*128+255):
((NUMENT-1)*128+128)
Entry NUMIDS-1: This field contains the last NVM Set Attributes Entry in the list, if
present.

Figure 318: NVM Set Attributes Entry
Bytes Description
01:00 NVM Set Identifier  (NSETID): This field indicates the identifier of the NVM Set in the NVM
subsystem that is described by this entry.
03:02 Endurance Group Identifier  (ENDGID): This field indicates the Endurance Group for this NVM
Set. Refer to section 3.2.3.
07:04 Reserved
11:08
Random 4 KiB Read Typical  (R4KRT): This field indicates the typical time to complete a 4  KiB
random read in 100 nanosecond units  when the NVM Set is in a Predictable Latency Mode
Deterministic Window and there is 1 outstanding command per NVM Set.
15:12
Optimal Write Size (OWS): This field indicates the size in bytes for optimal write performance.  A
value of 0h indicates that no Optimal Write Size is specified. This field should be cleared to 0h when
namespaces within an NVM Set have different User Data F ormats that do not allow an Optimal
Write Size to be specified.
31:16 Total NVM Set Capacity (TNVMSC): This field indicates the total NVM capacity in this NVM Set.
The value is in bytes.
47:32 Unallocated NVM Set Capacity (UNVMSC): This field indicates the unallocated NVM capacity in
this NVM Set. The value is in bytes.
127:48 Reserved
5.1.13.2.5 I/O Command Set specific Identify Namespace data structure (CNS 05h)
An I/O Command Set specific Identify Namespace data structure (refer to the applicable I/O Command Set
specification) is returned to the host for the specified namespace if the value in the Namespace Identifier
(NSID) field is an active NSID. If the value in the NSID field specifies an inactive NSID, then  the controller
returns a zero filled data structure.
The specific Identify Namespace data structure that is returned by this command is specified by the
Command Set Identifier (CSI) field (refer to Figure 311). If the I/O Command Set associated with the
specified namespace does not support the Identify Namespace data structure specified by the CSI field ,
then the controller shall abort the command with a status code of Invalid Field in Command.
