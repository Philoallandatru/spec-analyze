---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 75
headings: []
---

# Source page 75

NVM Express® Base Specification, Revision 2.1

53
Figure 36: Offset 0h: CAP – Controller Capabilities
Bits Type Reset Description
55:52 RO Impl
Spec
Memory Page Size Maximum (MPSMAX): This field indicates the maximum host
memory page size that the controller supports. The maximum memory page size
is (2 ^ (12 + MPSMAX)). The host shall not configure a memory page size in
CC.MPS that is larger than this value.
For Discovery controllers this field shall be cleared to 0h.
51:48 RO Impl
Spec
Memory Page Size Minimum (MPSMIN): This field indicates the minimum host
memory page size that the controller supports. The minimum memory page size is
(2 ^ (12 + MPSMIN)). The host shall not configure a memory page size in CC.MPS
that is smaller than this value.
For Discovery controllers this shall be cleared to 0h.
47:46 RO Impl
Spec
Controller Power Scope (CPS): This field indicates scope of controlling the main
power for this controller.
Value Power Scope
00b Not Reported
01b Controller scope
10b Domain scope (i.e ., the NVM subsystem supports multiple domains
(refer to section 3.2.5).
11b NVM subsystem scope (i.e., the NVM subsystem does not support
multiple domains).
If the NSSS bit is set to ‘1’, then this field shall not be cleared to 00b.
45 RO Impl
Spec
Boot Partition Support (BPS): This bit indicates whether the controller supports
Boot Partitions. If this bit is set to '1‘, then the controller supports Boot Partitions. If
this bit is cleared to ‘0‘, then the controller does not support Boot Partitions. Refer
to section 8.1.3.
44:37 RO Impl
Spec
Command Sets Supported (CSS): This field indicates the I/O Command Set(s)
that the controller supports.
For Discovery controllers, this field should have only the NCSS bit set to 1.
Bits Description
7
No I/O Command Set Support (NOIOCSS): This bit indicates whether
no I/O Command Set is supported (i.e., only the Admin Command Set
is supported).
This bit shall be set to ‘1’ if no I/O Command Set is supported.
6
I/O Command Set Support ( IOCSS): This bit indicates whether the
controller supports one or more I/O Command Sets and supports the
Identify I/O Command Set data structure (refer to section 5.1.13.2.19).
Controllers that support I/O Command Sets other than the NVM
Command Set shall set this bit to ‘1’. Controllers that only support the
NVM Command Set may set this bit to ‘1’.
5:1 Reserved
0 NVM Command Set Support (NCSS):  This bit indicates whether the
controller supports the NVM Command Set or a Discovery controller.

36 RO Impl
Spec
NVM Subsystem Reset Supported (NSSRS): This bit indicates whether the
controller supports the NVM Subsystem Reset feature defined in section 3.7.1. This
bit is set to '1' if the controller supports the NVM Subsystem Reset feature. This bit
is cleared to ‘0' if the controller does not support the NVM Subsystem Reset feature.
For Discovery controllers, this field shall be cleared to 0h.
