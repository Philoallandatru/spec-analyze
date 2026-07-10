---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 344
headings: []
---

# Source page 344

NVM Express® Base Specification, Revision 2.1

322
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
525 M M R
Volatile Write Cache (VWC): This field indicates attributes related to the presence of a
volatile write cache in the controller.
Bits Description
07:03 Reserved
02:01
Flush Behavior (FB): This field indicates Flush command behavior (refer
to section 7.2) if the NSID value is set to FFFFFFFFh as follows:
Value Definition
00b
Support for the NSID field set to FFFFFFFFh is not indicated.
Only controllers compliant with NVM Express Base
Specification revision 1.3 and earlier shall be allowed to return
this value.
01b Reserved
10b
The Flush command does not support the NSID field set to
FFFFFFFFh. The controller shall fail a Flush command with the
NSID set to FFFFFFFFh with a status code of Invalid
Namespace or Format.
11b The Flush command supports the NSID field set to
FFFFFFFFh.

00
Volatile Write Cache Present (VWCP): If this bit is set to ‘1’, then a volatile
write cache is present in the controller. If this bit is cleared to ‘0’ , then a
volatile write cache is not present in the controller.
If a volatile write cache is present, then the host controls whether the volatile
write cache is enabled with a Set Features command specifying the Volatile
Write Cache feature identifier (refer to section 5.1.25.1.4). The Flush
command (refer to section 7.2) is used to request that the contents of a
volatile write cache be made non-volatile.

527:526 M R R
Atomic Write Unit Normal (AWUN):  This field is specific to namespaces that are
associated with command sets that specify logical blocks (i.e., Command Set Identifier
0h or 2h), and shall be cleared to 0h for namespaces that are not associated with
command sets that specify logical blocks . Refer to the applicable I/O Command Set
specification (e.g., the Atomic Operation section of the NVM Command Set Specification).
529:528 M M R
Atomic Write Unit Power Fail (AWUPF): This field is specific to namespaces that are
associated with command sets that specify logical blocks (i.e., Command Set Identifier
0h or 2h), and shall be cleared to 0h for namespaces that are not associated with
command sets that specify logical blocks.  Refer to the applicable I/O Command Set
specification (e.g., the Atomic Operation section of the NVM Command Set Specification).
530 M M R
I/O Command Set Vendor Specific Command Configuration (ICSVSCC): This field
indicates the configuration settings for vendor specific I/O  command handling. Refer to
section 8.1.26.
Bits Description
7:1 Reserved
0
Same NVM Vendor Specific Command Format (SNVSCF): If this bit is set
to ‘1’, then all vendor specific I/O commands use the format defined in Figure
93. If this bit is cleared to ‘0’, then the format of all vendor specific I/O
commands is vendor specific.
