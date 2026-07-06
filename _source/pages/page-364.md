---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 364
headings: ["5.1.13.2.17 I/O Command Set specific Allocated Namespace ID list (CNS 1Ah)", "5.1.13.2.18 I/O Command Set specific Identify Namespace data structure for an Allocated Namespace ID (CNS 1Bh)", "5.1.13.2.19 Identify I/O Command Set data structure (CNS 1Ch)"]
---

# Source page 364

NVM Express® Base Specification, Revision 2.1

342
5.1.13.2.17 I/O Command Set specific Allocated Namespace ID list (CNS 1Ah)
A list of up to 1,024 namespace IDs is returned to the host containing allocated NSIDs in increasing order
that are greater than the value specified in the Namespace Identifier (NSID) field of the Identify command
and as specified by the Command Set Identifier (CSI) field of th e command. Only NSIDs for namespaces
associated with the I/O Command Set specified in CSI are returned . For CSI values not supported by the
controller the command is aborted with a status code of Invalid Field in Command.
The controller should abort the command with a status code of Invalid Namespace or Format if the NSID
field is set to FFFFFFFEh or FFFFFFFFh. The NSID field may be cleared to 0h to retrieve a Namespace
List including the namespace starting with NSID of 1h. The data structure returned is a Namespace List
(refer to section 4.6.2).
5.1.13.2.18 I/O Command Set specific Identify Namespace data structure for an Allocated
Namespace ID (CNS 1Bh)
An I/O Command Set specific Identify Namespace data structure (refer to section 5.1.13.2.5) is returned to
the host for the specified namespace if the value in the Namespace Identifier (NSID) field is an allocated
NSID. If the value in the NSID field is an unallocated NSID , then the controller returns a zero filled data
structure.
The specific Identify Namespace data structure that is returned by this command is specified by the
Command Set Identifier (CSI) field in the command (refer to Figure 311). If the I/O Command Set associated
with the specified namespace does not support the specific Identify Namespace data structure specified by
the CSI field, then the controller shall abort the command with a status code of Invalid Field in Command.
If the value in the NSID field is an invalid NSID, then the controller shall abort the command with a status
code of Invalid Namespace or Format. If the NSID field is set to FFFFFFFFh, then  the controller should
abort the command with a status code of Invalid Namespace or Format.
5.1.13.2.19 Identify I/O Command Set data structure (CNS 1Ch)
The Identify I/O Command Set data structure (refer to Figure 327) is returned to the host for the controller
specified in the Controller ID (CNTID) field of the command if the CNTID field does not have a value of
FFFFh. If the CNTID field has a value of FFFFh, then the Identify I/O Command Set data structure is
returned to the host for the controller processing the command.
This CNS value shall be implemented if the CAP.CSS.IOCSS bit is set to ‘1’.
The Identify I/O Command Set data structure consists of an array of I/O Command Set Vectors (refer to
Figure 328) that describe the I/O Command Sets that the controller supports and the combination of
supported I/O Command Sets that may be simultaneously used. The I/O Command Set Profile Feature
value indicates the index of the I/O Command Set Combination that is cu rrently selected (refer to section
5.1.25.1.17). I/O Command Set Combination 0 has an index value of 0h, I/O Command Set Combination 1
has an index value of 1h, and so on. Only I/O Command Sets that have a bit set to ‘1’ in the I/O Command
Set Vector of the I/O Command Set Combination selected by the I/O Command Set Profile Feature value
may be used. All other I/O Command Sets are treated as unsupported I/O Command Sets.
Figure 327: Identify I/O Command Set Data Structure
Bytes Description
7:0
I/O Command Set Combination 0  (IOCSC0): This field contains an I/O Command Set Vector
indicating the first I/O Command Set or combination of I/O Command Sets that are simultaneously
supported. If only one I/O Command Set is supported, then this field has only one bit set.
15:8
I/O Command Set Combination 1  (IOCSC1): This field contains an I/O Command Set Vector
indicating the second I/O Command Set or combination of I/O Command Sets that are simultaneously
supported if a second I/O Command Set combination is supported; otherwise, this field is cleared to
0h.
If this field is cleared to 0h, then no further I/O Command Set combinations are supported and
subsequent I/O Command Set Combinations shall have a value of 0h.
