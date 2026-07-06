---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 77
headings: ["3.1.4.2 Offset 8h: VS – Version"]
---

# Source page 77

NVM Express® Base Specification, Revision 2.1

55
Figure 36: Offset 0h: CAP – Controller Capabilities
Bits Type Reset Description
18:17 RO Impl
Spec
Arbitration Mechanism Supported (AMS): This field is bit significant and
indicates the optional arbitration mechanisms supported by the controller. If a bit is
set to ‘1’, then the corresponding arbitration mechanism is supported by the
controller. Refer to section 3.4.4 for arbitration details.
Bits Description
1 Vendor Specific (VS): Vendor Specific arbitration mechanism.
0
Weighted Round Robin with Urgent Priority Class (WRRUPC):
Weighted Round Robin with Urgent Priority Class  arbitration
mechanism.
The round robin arbitration mechanism is not listed since all controllers shall
support this arbitration mechanism.
For Discovery controllers, this property shall be cleared to 0h.
16 RO Impl
Spec
Contiguous Queues Required (CQR): This bit is set to ‘1’ if the controller requires
that I/O Submission Queues and I/O Completion Queues are required to be
physically contiguous. This bit is cleared to ‘0’ if the controller supports I/O
Submission Queues and I/O Completion Queues that are not physically
contiguous. If this bit is set to ‘1’, then the Physically Contiguous bit (CDW11.PC)
in the Create I/O Submission Queue and Create I/O Completion Queue commands
shall be set to ‘1’.
For controllers using a message -based transport, this property shall be set to a
value of 1.
15:00 RO Impl
Spec
Maximum Queue Entries Supported (MQES): This field indicates the maximum
individual queue size that the controller supports. For NVMe over PCIe
implementations, this value applies to the I/O Submission Queues and I/O
Completion Queues that the host creates. For NVMe over Fabrics implementations,
this value applies to only the I/O Submission Queues that the host creates. This is
a 0’s based value. The minimum value is 1h, indicating two entries.
 Offset 8h: VS – Version
This property is Read Only (RO) and indicates the version of this specification that the controller supports,
as defined in Figure 37.
Figure 37: Specification Version Descriptor
Bits Description
31:16 Major Version (MJR): An integer value indicating the major version number of this specification which is
supported by the controller.
15:08 Minor Version (MNR): An integer value indicating the minor version number of this specification which is
supported by the controller.
07:00
Tertiary Version (TER): An integer value indicating the tertiary version number of this specification which
is supported by the controller. If this field is cleared to 0h, then this specification does not have a tertiary
version number.
The reset value for each field is described in Figure 38:
Figure 38: NVM Express Base Specification Version Property
Reset Values
Specification
Version 1 MJR Field MNR Field TER Field
1.0 1h 0h 0h
