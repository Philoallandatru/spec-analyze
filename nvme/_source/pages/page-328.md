---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 328
headings: []
---

# Source page 328

NVM Express® Base Specification, Revision 2.1

306
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
111 M M M
Controller Type (CNTRLTYPE):  This field specifies the controller type. A value of 0h
indicates that the controller type is not reported.
Implementations compliant with NVM Express Base Specificatio n, Revision 1.4 or later
shall report a controller type (i.e., the value 0h is reserved and shall not be used).
Implementations compliant with an earlier specification version may report a value of 0h
to indicate that a controller type is not reported.
Value Controller Type
0h Reserved (controller type not reported)
1h I/O controller
2h Discovery controller
3h Administrative controller
4h to FFh Reserved

127:112 O O R
FRU Globally Unique Identifier (FGUID):  This field contains a 128 -bit value that is
globally unique for a given Field Replaceable Unit (FRU). Refer to the NVM Express®
Management Interface Specification for the definition of a FRU. This field remains fixed
throughout the life of the FRU. This field shall contain the same value for each controller
associated with a given FRU.
This field uses the EUI -64 based 16-byte designator format. Bytes 122:120 contain the
24-bit Organizationally Unique Identifier (OUI) value assigned by the IEEE Registration
Authority. Bytes 127:123 contain an extension identifier assigned by the correspon ding
organization. Bytes 119:112 contain the vendor specific extension identifier assigned by
the corresponding organization. Refer to the IEEE EUI -64 guidelines for more
information. This field is big endian (refer to section 4.5.4).
When not implemented, this field contains a value of 0h.
129:128 O O R
Command Retry Delay Time 1 (CRDT1): If the Do Not Retry (DNR) bit is cleared to ‘0’
in the CQE and the Command Retry Delay (CRD) field is set to 01b in the CQE, then this
value indicates the command retry delay time in units of 100 milliseconds.
131:130 O O R
Command Retry Delay Time 2 (CRDT2): If the DNR bit is cleared to ‘0’ in the CQE and
the CRD field is set to 10b in the CQE, then this value indicates the command retry delay
time in units of 100 milliseconds.
133:132 O O R
Command Retry Delay Time 3 (CRDT3): If the DNR bit is cleared to ‘0’ in the CQE and
CRD field is set to 11b in the CQE, then this value indicates the command retry delay
time in units of 100 milliseconds.
134 O R R
Controller Reachability Capabilities (CRCAP):  This field specifies reachability
capabilities of the controller and NVM subsystem.

Bits Description
7:2 Reserved
1
Reachability Group ID Changeable (RGIDC) : If this bit is set to ‘1’, then
the RGRPID field in the I/O Command Set Independent Identify Namespace
data structure (refer to Figure 319) does not change while the namespace
is attached to any controller. If this bit is cleared to ‘0’, then the RGRPID field
may change while the namespace is attached to any controller. Refer to
section 8.1.19.
0
Reachability Reporting Supported (RRSUP) : If this bit is set to ‘1’, then
the NVM subsystem supports Reachability Reporting (refer to section
8.1.19). If this bit is cleared to ‘0’, then the NVM subsystem does not support
Reachability Reporting.
239:135    Reserved
252:240    Reserved for the NVMe Management Interface.
