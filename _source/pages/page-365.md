---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 365
headings: ["5.1.13.2.20 I/O Command Set Independent Identify Namespace data structure for an Allocated Namespace ID (CNS 1Fh)", "5.1.13.2.21 Supported Controller State Formats (CNS 20h)"]
---

# Source page 365

NVM Express® Base Specification, Revision 2.1

343
Figure 327: Identify I/O Command Set Data Structure
Bytes Description
… …
4095:4088
I/O Command Set Combination 511 (IOCSC511): This field contains an I/O Command Set Vector
indicating the 511th I/O Command Set or combination of I/O Command Sets that are simultaneously
supported if a 511th I/O Command Set combination is supported; otherwise, this field is cleared to 0h.

Figure 328: I/O Command Set Vector
Bits Description
63:5 Reserved
4
Computational Programs Namespace Command Set (CPNCS): This bit is set to ‘1’ if the Computational
Programs Command Set is selected. This bit is cleared to ‘0’ if the Zoned Namespace Command Set is not
selected.
3
Subsystem Local Memory Command Set (SLMCS): This bit is set to ‘1’ if the Subsystem Local Memory
Command Set is selected. This bit is cleared to ‘0’ if the Subsystem Local Memory Command Set is not
selected.
2 Zoned Namespace Command Set (ZNSCS): This bit is set to ‘1’ if the Zoned Namespace Command Set
is selected. This bit is cleared to ‘0’ if the Zoned Namespace Command Set is not selected.
1 Key Value Command Set (KVCS): This bit is set to ‘1’ if the Key Value Command Set is selected. This bit
is cleared to ‘0’ if the Key Value Command Set is not selected.
0 NVM Command Set (NVMCS): This bit is set to ‘1’ if the NVM Command Set is selected. This bit is cleared
to ‘0’ if the NVM Command Set is not selected.
5.1.13.2.20 I/O Command Set Independent Identify Namespace data structure for an Allocated
Namespace ID (CNS 1Fh)
An I/O Command Set Independent Identify Namespace data structure (refer to section 5.1.13.2.8) is
returned to the host for the namespace specified in the Namespace Identifier (NSID) field if it is an allocated
NSID. If the specified NSID is an unallocated NSID, then the controller returns a zero filled data structure.
If the specified NSID is an invalid NSID, then the controller shall abort the command with a status code of
Invalid Namespace or Format. If the NSID field is set to FFFFFFFFh, then the controller shall abort the
command with a status code of Invalid Namespace or Format.
5.1.13.2.21 Supported Controller State Formats (CNS 20h)
A Supported Controller State Formats data structure (refer to Figure 329) is returned to the host identifying
the supported NVMe Controller State data structures (refer to Figure 330) and a list of UUIDs that identifies
the vendor specific controller state information.
A host uses an index into each list to identify the format and information returned by a Migration Receive
command specifying the Get Controller State management operation (refer to section  5.1.16.1.1).
A host uses an index into each list to identify the format and information contained in a Migration Send
command specifying the Set Controller State management operation of the Migration Send command (refer
to section 5.1.17.1.3).
Figure 329: Supported Controller State Formats Data Structure
Bytes Description
Header
0
Number of Versions (NV): This field indicates the number of entries in the NVMe Controller
State Version list. A value of 0h indicates that no entries exist in the NVMe Controller State
Version list.
1
Number of UUIDs (NUUID): This field indicates the number of entries in the Vendor Specific
Controller State UUID Supported list. A value of 0h indicates that no entries exist in the Vendor
Specific Controller State UUID Supported List.
