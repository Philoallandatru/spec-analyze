---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 160
headings: ["4.2  Completion Queue Entry", "4.2.1 Admin Command and I/O Command Common CQE"]
---

# Source page 160

NVM Express® Base Specification, Revision 2.1

138
Figure 94: Fabrics Command – Submission Queue Entry Format
Bytes Description
63:40 Fabrics Command Type Specific (FCTS): This field is Fabrics command type specific.

Figure 95: Fabrics Command – Command Dword 0
Bits Description
31:16 Command Identifier (CID): Refer to Figure 91.
15:14
PRP or SGL for Data Transfer (PSDT):
Value Description
00b
No Data Transfered: This value is used for Fabrics commands that do not transfer data.
If this value is used for Fabrics commands that transfer data, then SGLs are used for this
transfer. A host should use the value 10b rather than 00b for Fabrics commands that transfer
data.
01b Reserved
10b Data Transferred: This value is used for Fabrics commands that transfer data. SGLs are
used for this transfer.
11b Reserved

13:10 Reserved
09:08 Fused Operation (FUSE): Refer to Figure 91. There are no fused Fabrics commands and as a result
this field is cleared to 00b.
07:00 Opcode (OPC): This field is set to 7Fh to specify a Fabrics command.
4.2  Completion Queue Entry
 Admin Command and I/O Command Common CQE
The Common Completion Queue Entry Layout is at least 16 bytes in size. Figure 96 describes the layout
of the first 16 bytes of the completion queue entry data structure which follows the Common Completion
Queue Entry Layout. The contents of Dword 0 and Dword 1 are command specific. If a command uses
Dword 0 or Dword 1, then the defini tion of these dwords is contained within the associated command
definition. If a command does not use Dword 0 or Dword 1, then the unused field(s) are reserved. Dword 2
is defined in Figure 97 and Dword 3 is defined in Figure 98.
If a completion queue entry is constructed via multiple writes, the Phase Tag bit shall be updated in the last
write of that completion queue entry.
Figure 96: Common Completion Queue Entry Layout – Admin and All I/O Command Sets
 31 23 15 7 0
DW0 Command Specific
DW1 Command Specific
DW2 SQ Identifier SQ Head Pointer
DW3 Status P Command Identifier

Figure 97: Completion Queue Entry: DW 2
Bits Description
31:16
SQ Identifier (SQID): Indicates the Submission Queue to which the associated command was issued.
This field is used by host software when more than one Submission Queue shares a single Completion
Queue to uniquely determine the command completed in combination with the Command Identifier (CID).
This is a reserved field in NVMe over Fabrics implementations.
