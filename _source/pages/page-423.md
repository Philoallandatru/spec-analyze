---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 423
headings: []
---

# Source page 423

NVM Express® Base Specification, Revision 2.1

401
section 8.1.22). However, reservations and registrations associated with a Host Identifier value of 0h do not
persist across a Controller Level Reset since a host that uses a Host Identifier value of 0h is treated as a
different host after a Controller Level Reset.
A Set Features command should be used to change a Host Identifier value of 0h to a non-zero value before
using streams (refer to section 8.1.8.3) or using reservations (refer to section 8.1.22). Information (i.e.,
streams or reservations) associated with a Host Identifier value of 0h retain the association to that Host
Identifier if the Host Identifier value is changed and are not associated with the host that has the non -zero
Host Identifier.
The NVM subsystem indicates if reservations are supported with a Host Identifier value of 0h with the RHII
bit in the Controller Attributes field of the Identify Controller data structure (refer to Figure 312). The NVM
subsystem indicates if streams are supported with a Host Identifier value of 0h with the SRNZID bit in the
NVM Subsystem Stream Capability field of the streams directive return parameters (refer to Figure 607).
The Host Identifier feature shall not be a saveable feature.
The requirements and use of the Host Identifier feature are dependent on whether the NVMe over PCIe
implementation or the NVMe over Fabrics implementation is supported. Refer to section 5.1.25.1.28.1 and
section 5.1.25.1.28.2.
Figure 437: Host Identifier – Command Dword 11
Bits Description
31:01 Reserved
00
Enable Extended Host Identifier (EXHID): If this bit is set to ‘1’, then the host is requesting the use of
an extended 128-bit Host Identifier. If this bit is cleared to ‘0’, then the host is requesting the use of a 64-
bit Host Identifier. NVMe over Fabrics implementations shall use an extended 128-bit Host Identifier.
If the controller does not support a 128-bit Host Identifier as indicated in the Controller Attributes field in
the Identify Controller data structure and the host sets this bit to ‘1’, then a status code of Invalid Field in
Command shall be returned.
If the controller does not support a 64-bit Host Identifier (e.g., the device is an NVMe over Fabrics device)
and the host clears this bit to ‘0’, then a status code of Invalid Field in Command shall be returned.
If the NVM subsystem supports a 64 -bit Host Identifier, supports a 128 -bit Host Identifier and detects
that another controller in the NVM subsystem is already using a non-zero Host Identifier of a different
size than the size requested in this command, then a status code of Host Identifier Inconsistent Format
shall be returned.

Figure 438: Host Identifier – Data Structure Entry
Bytes Description
15:00
Host Identifier (HOSTID): This field specifies a 64-bit or 128-bit identifier that uniquely identifies the host
associated with the controller within the NVM subsystem. The host provides an 8 byte or 16 byte data
structure depending on the value specified in the Enable Extended Host Identifier  bit. The value of the
Host Identifier used by a host, the method used to select this value, and the method used to ensure
uniqueness are outside the scope of this specif ication. Controllers in an NVM s ubsystem that have the
same Host Identifier are assumed to be associated with the s ame host and have the same reservation
and registration rights.
If the current Host Identifier value is cleared to 0h, then the current Host Identifier shall be set to the value
specified in this field.
If the current Host Identifier value is set to a non-zero value, then the controller shall abort the command
with a status code of Command Sequence Error.
A Host Identifier value of 0h indicates that the host is not associated with any other controller in the NVM
subsystem.
