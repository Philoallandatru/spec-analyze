---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 432
headings: ["5.1.25.3 Message-Based Transport Feature Specific Information (Fabrics)"]
---

# Source page 432

NVM Express® Base Specification, Revision 2.1

410
Each Host Memory Buffer Descriptor Entry shall describe a host memory address in memory page size
units and the number of contiguous memory page size units associated with the host address.
Figure 453: Host Memory Buffer – Host Memory Buffer Descriptor Entry
Bits Description
127:96 Reserved
95:64 Buffer Size (BSIZE): Indicates the number of contiguous memory page size (CC.MPS) units for this
descriptor. If this field is cleared to 0h, then the controller shall ignore this descriptor.
63:00
Buffer Address (BADD): Indicates the host memory address for this descriptor aligned to the
memory page size (CC.MPS). The least significant bits (n:0) of this field indicate the offset within the
memory page is 0h (e.g., if the memory page size is 4 KiB, then bits 11:00 shall be 0h; if the memory
page size is 8 KiB, then bits 12:00 shall be 0h).
If a Get Features command is issued for this Feature, the attributes specified in Figure 454 are returned in
Dword 0 of the completion queue entry and the Host Memory Buffer Attributes data structure, whose
structure is defined in Figure 455, is returned in the data buffer for that command.
Figure 454: Host Memory Buffer – Completion Queue Entry Dword 0
Bits Description
31:04 Reserved
03
Host Memory Non-operational Access Restricted (HMNAR): If this bit is set to ‘1’, then HMB non-
operational power state access restriction is currently restricting the controller from accessing the
HMB. If this bit is cleared to ‘0’, then HMB non -operational power state access restriction is not
currently restricting the controller from accessing the HMB (e.g., the HMB is not enabled, the controller
is currently in an operational power state, HMB non-operational power state access restriction is not
supported, or HMB non-operational power state access restriction is not enabled).
02
Host Memory Non-operational Access Restriction Enable (HMNARE): If this bit is set to ‘1’, then
HMB non-operational power state access restriction is enabled (refer to the HMNARE bit description
in Figure 447). If this bit is cleared to ‘0’, then HMB non-operational power state access restriction is
not enabled.
01 Not Used (NUSED): This bit is not used for a Get Feature command and shall be cleared to ‘0’.
00
Enable Host Memory (EHM): If this bit is set to ‘1’, then the host memory buffer is enabled and the
controller may use the host memory buffer. If this bit is cleared to ‘0’, then the host memory buffer is
disabled, and the controller is not using the host memory buffer.

Figure 455: Host Memory Buffer – Attributes Data Structure
Bytes Description
3:0 Host Memory Buffer Size (HSIZE): This field indicates the size of the host memory buffer allocated
in memory page size units.
7:4
Host Memory Descriptor List Address Lower (HMDLAL): This field indicates the least significant
32 bits of the physical location of the Host Memory Descriptor List (refer to Figure 452) for the host
memory buffer. This address shall be 16 byte aligned. The least significant 4 bits shall be cleared to
‘0’.
11:8
Host Memory Descriptor List Address Upper (HMDLAU): This field indicates the most significant
32 bits of the physical location of the Host Memory Descriptor List (refer to Figure 452) for the host
memory buffer.
15:12
Host Memory Descriptor List Entry Count (HMDLEC): This field indicates the number of valid Host
Memory Descriptor Entries (refer to Figure 453) in the Host Memory Descriptor List (refer to Figure
452).
4095:16 Reserved
 Message-Based Transport Feature Specific Information (Fabrics)
There are no features that are specific to the Message-based transport model.
