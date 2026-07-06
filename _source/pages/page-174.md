---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 174
headings: []
---

# Source page 174

NVM Express® Base Specification, Revision 2.1

152
Figure 110: PRP Entry – Page Base Address and Offset
Bits Description
63:00
Page Base Address and Offset (PBAO): This field indicates the 64 -bit physical memory page
address. The least significant bits (n:0) of this field indicate the offset within the memory page (e.g., if
the memory page size is 4 KiB, then bits 11:00 form the Offset; if the memory page size is 8 KiB, then
bits 12:00 form the Offset). If this entry is not the first PRP entry in the command or a PRP List pointer
in a command, then the Offset portion of this field shall be cleared to 0h. The Offset shall be dword
aligned, indicated by bits 1:0 being cleared to 00b.
Note: The controller is not required to check that bits 1:0 are cleared to 00b. The controller may report
an error of PRP Offset Invalid if bits 1:0 are not cleared to 00b. If the controller does not report an
error of PRP Offset Invalid, then the controller shall operate as if bits 1:0 are cleared to 00b.
A physical region page list (PRP List) is a set of PRP entries in a single page of contiguous memory. A PRP
List describes additional PRP entries that could not be described within the command itself. Any PRP
entries described within the command are not du plicated in a PRP List. If the amount of data to transfer
requires multiple PRP List memory pages, then the last PRP entry before the end of the memory page shall
be a pointer to the next PRP List, indicating the next segment of the PRP List. Figure 111 shows the layout
of a PRP List where each PRP entry identifies memory pages that are physically contiguous. Figure 112
shows the layout of a PRP List where each PRP entry identifies a different memory page (i.e., the memory
pages are not physically contiguous).
Figure 111: PRP List Layout for Physically Contiguous Memory Pages
63      n+1 n  0
Page Base Address p 0h
Page Base Address p+1 0h
…
Page Base Address p+q 0h
Page Base Address p+q+1 0h

Figure 112: PRP List Layout for Physically Non-Contiguous Memory Pages
63      n+1 n  0
Page Base Address p 0h
Page Base Address q 0h
…
Page Base Address r 0h
Page Base Address s 0h
Dependent on the command definition, the first PRP entry contained within the command may have a non-
zero offset within the memory page. The first PRP List entry (i.e., the first pointer to a memory page
containing additional PRP entries) that if present is typically contained in the PRP Entry 2 locati on within
the command, shall be qword aligned and may also have a non-zero offset within the memory page.
PRP entries contained within a PRP List shall have a memory page offset of 0h. If a second PRP entry is
present within a command, it shall have a memory page offset of 0h. In both cases, the entries are memory
page aligned based on the value in CC.MPS. If the controller receives a non -zero offset for these PRP
entries the controller should return an error of PRP Offset Invalid.
PRP Lists shall be minimally sized with packed entries starting with entry 0. If more PRP List pages are
required, then the last entry of the PRP List contains the Page Base Address of the next PRP List page.
The next PRP List page shall be memory page ali gned. The total number of PRP entries required by a
command is implied by the command parameters and memory page size.
