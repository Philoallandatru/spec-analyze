---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 439
headings: []
---

# Source page 439

NVM Express® Base Specification, Revision 2.1

417
Figure 468: Track Memory Changes Data Structure
Bytes Description
Header
00
Version (VER): This field specifies the version of this data structure.
If this field is not cleared to 0h, then the controller shall abort this command with a status code
of Invalid Field in Command.
02:01 Reserved
03
Requested Memory Range Tracking Granularity (RMRTG):  This field specifies the
granularity for the Length field in each Memory Range Tracking Descriptor contained in this
data structure. The value specified is a power of two (2^n) times 4 KiB. For example, a value
of 8h indicates that the granularity of tracking is (2^8) * 4 KiB which in 1 MiB.
07:04
Number of Memory Range Tracking Descriptors (RNMRTD):  This field specifies the
number of Memory Range Tracking Descriptors in the Memory Range Tracking Descriptor
list.
This field is a 1’s based number.
Memory Range Tracking Descriptor List
19:08 Memory Range Tracking Descriptor 0: This field contains the first Memory Range Tracking
Descriptor as defined in Figure 469.
31:20 Memory Range Tracking Descriptor 1: This field contains the second Memory Range
Tracking Descriptor as defined in Figure 469, if any.
…
(RNMRTD*12)+7:
(RNMRTD-1)*12+8
Memory Range Tracking Descriptor RNMR TD-1: This field contains the last Memory
Range Tracking Descriptor as defined in Figure 469, if any.
Figure 469 defines a Memory Range Tracking Descriptor. The address range specified by a Memory Range
Tracking Descriptor describes addresses specified in an SGL or PRP for a command submitted by a host
to the controller specified in the CNTLID field (refer to Figure 467).
Figure 469: Memory Range Tracking Descriptor
Bytes Description
07:00
Address (SADDR): This field specifies the starting 64-bit address of the memory range where the address
is host memory (refer to section 1.5.46) associated with the controller specified by the CNTLID field (refer
to Figure 467). The address is aligned to the granularity specified by the RMRTG field. For example, if the
value in the RMRTG field is 8h, then the value in this field is required to the be aligned to 1 MiB (i.e., bits
19:0 are cleared to 0h).
11:08
Length (LEN): This field specifies the length of this host memory range in units of the tracking granularity
as defined by RMRTG field. For example, if the value n the RMRTG field is 8h and the value of this field is
4h, then the length of this memory range is 1 MiB * 4h which is 4 MiB.
If the controller is unable to track the requested number of Memory Range Tracking Descriptors (i.e. ,
specified in the RNMRTD field) or unable to track the memory changes at the specified granularity (i.e.,
RMRTG field), then:
• the controller shall abort the command with a status code of Not Enough Resources; and
• Dword 0 and Dword 1 of the completion queue entry shall be returned as defined in Figure 470
and Figure 471.
If any Address field is not aligned to the granularity specified by the RMRTG field, then the controller shall
abort the command with a status code of Invalid Field in Command.
If the address range specified by the Address field and the Length field specifies an address that is not host
memory (refer to section 1.5.46) associated with the controller specified by the CNTLID field, then the
controller processing the command shall abort the command with a status code of Invalid Field in
Command. If:
• the command is successful; and
