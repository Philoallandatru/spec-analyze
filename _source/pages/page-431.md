---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 431
headings: []
---

# Source page 431

NVM Express® Base Specification, Revision 2.1

409
Figure 447: Host Memory Buffer – Command Dword 11
Bits Description
00
Enable Host Memory (EHM): If this bit is set to ‘1’, then the host memory buffer shall be enabled and
the controller may use the host memory buffer. If this bit is cleared to ‘0’, then the host memory buffer
shall be disabled, and the controller shall not use the host memory buffer.
If a Set Features command is processed with this bit cleared to ‘0’, then the controller shall ignore
Command Dword 12, Command Dword 13, Command Dword 14, and Command Dword 15.

Figure 448: Host Memory Buffer – Command Dword 12
Bits Description
31:00 Host Memory Buffer Size (HSIZE):  This field specifies the size of the host memory buffer allocated in
memory page size (CC.MPS) units.

Figure 449: Host Memory Buffer– Command Dword 13
Bits Description
31:00
Host Memory Descriptor List Lower Address (HMDLLA): This field specifies the least significant
32 bits of the physical location of the Host Memory Descriptor List (refer to Figure 452) for the Host
Memory Buffer. This address shall be 16 byte aligned, indicated by bits 3:0 being cleared to 0h.
Note: The controller shall operate as if bits 3:0 are cleared to 0h. However, the controller is not
required to check that bits 3:0 are cleared to 0h.

Figure 450: Host Memory Buffer – Command Dword 14
Bits Description
31:00 Host Memory Descriptor List Upper Address (HMDLUA): This field specifies the most significant 32
bits of the physical location of the Host Memory Descriptor List for the Host Memory Buffer.
The Host Memory Descriptor List Address (HMDLLA/HMDLUA) specifies the address of a physically
contiguous data structure in host memory that describes the address and length pairs of the Host Memory
Buffer. The number of address and length pairs is specified in the Host Memory Descriptor List Entry Count
in Figure 451. The Host Memory Descriptor List is described in Figure 452.
Figure 451: Host Memory Buffer – Command Dword 15
Bits Description
31:00 Host Memory Descriptor List Entry Count (HMDLEC): This field specifies the number of entries
provided in the Host Memory Descriptor List.
If the host specifies the Host Memory Descriptor List Entry Count field cleared to 0h, then the controller
shall abort the command with a status code of Invalid Field in Command.
Figure 452: Host Memory Buffer – Host Memory Descriptor List
Bytes Description
Host Memory Descriptor List
15:0 Host Memory Buffer Descriptor Entry 0 : This field is the first Host Memory Buffer
Descriptor (refer to Figure 453) in the list, if any.
31:16 Host Memory Buffer Descriptor Entry 1 : This field is the second Host Memory Buffer
Descriptor in the list, if any.
…
16*n+15:16*n Host Memory Buffer Descriptor Entry n : This field is the last Host Memory Buffer
Descriptor in the list where n = HMDLEC - 1 (refer to Figure 451), if any.
