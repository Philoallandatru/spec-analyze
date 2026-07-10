---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 349
headings: []
---

# Source page 349

NVM Express® Base Specification, Revision 2.1

327
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
1023:768 M M R
NVM Subsystem NVMe Qualified Name (SUBNQN):  This field specifies the NVM
Subsystem NVMe Qualified Name as a UTF-8 null-terminated string. Refer to section 4.7
for the definition of NVMe Qualified Name.
Support for this field is mandatory if the controller supports revision 1.2.1 or later as
indicated in the Version property (refer to section 3.1.4.2).
For a Discovery controller, if the Discovery subsystem containing the Discovery controller
has a unique Discovery Service NQN, then this field shall be set to that unique Discovery
Service NQN. If the Discovery subsystem containing the Discovery controller  does not
have a unique Discovery Service NQN, then this field shall be set to the well -known
Discovery Service NQN (i.e., nqn.2014-08.org.nvmexpress.discovery).
1791:1024    Reserved
Fabric Specific
1795:1792 M2 R R
I/O Queue Command Capsule Supported Size (IOCCSZ):  This field defines the
maximum I/O command capsule size in 16 byte units. The minimum value that shall be
indicated is 4 corresponding to 64 bytes.
1799:1796 M2 R R
I/O Queue Response Capsule Supported Size (IORCSZ): This field defines the
maximum I/O response capsule size in 16 byte units. The minimum value that shall be
indicated is 1 corresponding to 16 bytes.
1801:1800 M2 R R
In Capsule Data Offset (ICDOFF): This field defines the offset where data starts within
a capsule. This value is applicable to I/O Queues only (the Admin Queue shall use a
value of 0h).
The value is specified in 16 byte units. The offset is from the end of the submission queue
entry within the command capsule (starting at 64 bytes in the command capsule). The
minimum value is 0 and the maximum value is FFFFh.
1802 M2 M2 R
Fabrics Controller Attributes (FCATT): This field indicates attributes of the controller
that are specific to NVMe over Fabrics.
Bits Description
7:2 Reserved
1
Non-Zero NVM Set ID  Support (NZNSETIDS): If this bit is cleared to ‘0’,
then the NVM controller does not support a non-zero value in the NVMSETID
field in the Connect command (refer to section 6.3). If this bit is set to ‘1’,
then the NVM controller does support a non -zero value in the NVMSETID
field in the Connect command.
0
Dynamic Controller Model Support (DCMS): If this bit is cleared to ‘0’, then
the NVM subsystem uses a dynamic controller model. If this bit is set to ‘1’,
then the NVM subsystem uses a static controller model.

1803 M2 M2 R
Maximum SGL Data Block Descriptors (MSDBD): This field indicates the maximum
number of SGL Data Block or Keyed SGL Data Block descriptors that a host is allowed
to place in a capsule. A value of 0h indicates no limit.
1805:1804 M2 M2 R
Optional Fabric s Commands Support (OFCS): Indicate whether the controller
supports optional Fabrics commands.
Bits Description
15:1 Reserved
0
Disconnect Command Support (DCS): If this bit is cleared to ‘0’, then the
controller does not support the Disconnect command. If this bit is set to ‘1’,
then the controller supports the Disconnect command and deletion of
individual I/O Queues.
