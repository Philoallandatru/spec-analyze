---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 326
headings: []
---

# Source page 326

NVM Express® Base Specification, Revision 2.1

304
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
1
Non-Operational Power State Permissive Mode (NOPSPM): If this bit is
set to ‘1’, then the controller supports host control of whether the controller
may temporarily exceed the power of a non-operational power state for the
purpose of executing controller-initiated background operations in a non -
operational power state (i.e., Non -Operational Power State Permissive
Mode supported). If this bit is cleared to ‘0’, then the controller does not
support host control of whether the controller may exceed the power of a
non-operational state for the purpose of executing controller-initiated
background operations in a non -operational state (i.e., Non -Operational
Power State Permissive Mode not supported). Refer to section 5.1.25.1.10.
0
Host Identifier Support (HIDS): If this bit is set to ‘1’, then the controller
supports a 128 -bit Host Identifier. If this bit is  cleared to ‘0’, then the
controller does not support a 128-bit Host Identifier.

101:100 O O R
Read Recovery Levels Supported (RRLS): If Read Recovery Levels (RRL) are
supported, then this field shall be supported. If a bit is set to ‘1’, then the corresponding
Read Recovery Level is supported. If a bit is cleared to ‘0’, then the corresponding Read
Recovery Level is not supported.
Bits Description
15 Read Recovery Level 15 (RRL15): Fast Fail1
14 Read Recovery Level 14 (RRL14)
13 Read Recovery Level 13 (RRL13)
12 Read Recovery Level 12 (RRL12)
11 Read Recovery Level 11 (RRL11)
10 Read Recovery Level 10 (RRL10)
9 Read Recovery Level 9 (RRL9)
8 Read Recovery Level 8 (RRL8)
7 Read Recovery Level 7 (RRL7)
6 Read Recovery Level 6 (RRL6)
5 Read Recovery Level 5 (RRL5)
4 Read Recovery Level 4 (RRL4): Default1
3 Read Recovery Level 3 (RRL3)
2 Read Recovery Level 2 (RRL2)
1 Read Recovery Level 1 (RRL1)
0 Read Recovery Level 0 (RRL0)
Notes:
1. If Read Recovery Levels are supported, then this bit shall be set to ‘1’.
