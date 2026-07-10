---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 208
headings: []
---

# Source page 208

NVM Express® Base Specification, Revision 2.1

186
Figure 162: Create Controller Data Queue – PRP Entry 1
Bits Description
63:00
PRP Entry 1 (PRP1): If the PC bit is set to ‘1’, then this field specifies a 64 -bit base host memory (refer to
section 1.5.46) address pointer of the Controller Data Queue that is physically contiguous. The address
pointer is memory page aligned (based on the value in CC.MPS field) unless otherwise specified. If the PC
bit (refer to Figure 163) is cleared to ‘0’, then this field specifies a PRP List pointer that describes the list of
pages that constitute the Controller Data Queue. The list of pages is host memory (refer to section 1.5.46)
that is page aligned (based on the value in CC.MPS field) unless otherwise specified. In both cases the PRP
Entry shall have an offset of 0h. In a non -contiguous Controller Data Queue, each PRP Entry in the PRP
List shall have an offset of 0h. If there is a PRP Entry with a non-zero offset, then the controller shall return
an error of PRP Offset Invalid.
If the memory specified by this field is not host memory (e.g., CMB o r PMR), then the controller shall abort
the command with a status code of Invalid Field in Command.

Figure 163: Create Controller Data Queue – Command Dword 11
Bits Description
31:16
Create Queue Specific (CQS): The definition of this field is specific to the type of queue being created
(refer to Queue Type (QT) field in Figure 165). If the type of queue being created does not define the use of
this field, then this field is reserved.
15:01 Reserved
00
Physically Contiguous (PC): If set to ‘1’, then the Controller Data Queue is physically contiguous and PRP
Entry 1 (PRP1) is the address of a contiguous physical buffer. If cleared to ‘0’, then the Controller Data
Queue is not physically contiguous and PRP Entry 1 (PRP1) is a PRP List pointer.

Figure 164: Create Queue – Command Dword 12
Bits Description
31:00 Controller Data Queue Size (CDQSIZE): This field specifies the size of the queue to be created in dwords.
If the Controller Data Queue Size field value is not a multiple of the size of the entries for the type of
Controller Data Queue as specified by the Queue Type field (refer to Figure 165), then the controller shall
abort the command with a status code of Invalid Field in Command.
The Management Operation Specific field (refer to  Figure 161) for the Create Controller Data Queue
management operation is specified in Figure 165.
Figure 165: Create Controller Data Queue – Management Operation Specific
Bits Description
15:08 Reserved
07:00
Queue Type (QT): This field specifies the type of queue to be created which defines the information that
the controller posts into the entries of the queue.
Value Definition Reference Section
0h User Data Migration Queue 5.1.4.1.1.1
1h to BFh Reserved
C0h to FFh Vendor Specific

If the PRP Entry 1 field has a non-zero PRP offset, then the controller shall abort the command with a status
code of PRP Offset Invalid.
