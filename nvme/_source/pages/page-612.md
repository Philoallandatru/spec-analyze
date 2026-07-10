---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 612
headings: []
---

# Source page 612

NVM Express® Base Specification, Revision 2.1

590
have the host write the data and/or metadata to the Controller Memory Buffer rather than have the controller
fetch it from host memory.
The contents of the Controller Memory Buffer are undefined as the result of:
• the CMBMSC.CMSE bit transitioning from ‘0’ to ‘1’;
• a Controller Reset; or
• a Function Level Reset.
Host software should initialize any memory in the Controller Memory Buffer before being referenced (e.g.,
a Completion Queue shall be initialized by host software in order for the Phase Tag to be used correctly
(refer to section 4.2.4)).
A CMB implementation has a maximum sustained write throughput. The CMB implementation may also
have an optional write elasticity buffer used to buffer writes from CMB PCIe write requests. When the CMB
sustained write throughput is less than the PCI Express  link throughput, then such a write elasticity buffer
allows PCIe write request burst throughput to exceed the CMB sustained write throughput without back
pressuring into the PCI Express fabric.
The time required to transfer data from the write elasticity buffer to the CMB is the amount of data written
to the elasticity buffer divided by the Controller Memory Buffer Sustained Write Throughput (refer to section
3.1.4.19). The time to transfer the entire contents of the write elasticity buffer is the Controller Memory
Buffer Elasticity Buffer Size (refer to section 3.1.4.18) divided by the Controller Memory Buffer Sustained
Write Throughput.
A controller memory -based queue is used in the same manner as a host memory -based queue – the
difference is the memory address used is located within the controller’s own memory rather than in the host
memory. The Admin or I/O Queues may be placed in the C ontroller Memory Buffer. If the
CMBLOC.CQMMS bit (refer to Figure 47) is cleared to ‘0’, then for a particular queue, all memory
associated with it shall reside in either the Controller Memory Buffer or outside the Controller Memory
Buffer.
If the CMBLOC.CQPDS bit (refer to Figure 47) is cleared to ‘0’, then for all queues in the Controller Memory
Buffer, the queue shall be physically contiguous.
The controller may support PRP Lists and SGLs in the Controller Memory Buffer. If the CMBLOC.CDPMLS
bit (refer to Figure 47) is cleared to ‘0’, then for a particular PRP List or SGL associated with a single
command, all memory containing the PRP List or SGL shall be either entirely located in the Controller
Memory Buffer or entirely located outside the Controller Memory Buffer.
PRP Lists and SGLs associated with a command may be placed in the Controller Memory Buffer if that
command is present in a Submission Queue in the Controller Memory Buffer. If:
a) CMBLOC.CDPCILS bit (refer to Figure 47) is cleared to ‘0’; and
b) a command is not present in a Submission Queue in the Controller Memory Buffer,
then the PRP Lists and SGLs associated with that command shall not be placed in the Controller Memory
Buffer.
The controller may support data and metadata in the Controller Memory Buffer. If the CMBLOC.CDMMMS
bit (refer to Figure 47) is cleared to ‘0’, then all data and metadata, if any, associated with a particular
command shall be either entirely located in the Controller Memory Buffer or entirely located outside the
Controller Memory Buffer.
If the requirements for the Controller Memory Buffer use are violated by the host, the controller shall abort
the associated command with a status code of Invalid Use of Controller Memory Buffer.
The address region allocated for the CMB shall be 4  KiB aligned. It is recommended that a controller
allocate the CMB on an 8 KiB boundary. The controller shall support burst transactions up to the maximum
payload size, support byte enables, and arbitrary byte alignment. The host shall ensure that all writes to the
CMB that are needed for a command have been sent before updating the SQ Tail doorbell property. The
