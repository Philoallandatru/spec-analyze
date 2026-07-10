---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 341
headings: []
---

# Source page 341

NVM Express® Base Specification, Revision 2.1

319
521:520 M M R
Optional NVM Command Support (ONCS): This field indicates the optional I/O
commands and features supported by the controller. Refer to section 3.1.3.
If a controller supports more than one I/O Command Set, then the bits in the field
associated with command support reflect the aggregate support across all of the I/O
Command Sets supported or I/O Command Sets inherited from I/O Command Sets (e.g.,
Zoned N amespace Command Set). The Commands Supported and Effects log page
(refer to section 5.1.12.1.6) may be used to determine the support of commands for a
specific I/O Command Set.
Bits Description
15:13 Reserved
12
Namespace Zeroes Support (NSZS):  If this bit is set to ‘1’, then the
controller supports the Namespace Zeroes (NSZ) bit in the NVM Command
Set Write Zeroes command. If this bit is cleared to ‘0’, then the controller
does not support the Namespace Zeroes (NSZ) bit in the Write Zeroes
command. If the Write Zeroes command is not supported, then this bit shall
be cleared to ‘0’
11
Maximum Write Zeroes with Deallocate (MAXWZD): If this bit is set to ‘1’,
then the maximum data size for the NVM Command Set Write Zeroes
command depends on the value of the Deallocate bit in the Write Zeroes
command and the value in the WZDSL field in the I/O Command Set specific
Identify Controller data structure for the NVM Command Set (refer to the I/O
Command Set specific Identify Controller Data Structure (CNS 06h, CSI
00h) section in the NVM Command Set Specification). If this bit is cleared to
‘0’, then this bit has no effect. If the Write Zeroes command is not supported
or the WZSL field in that data structure is cleared to 0h, then this bit shall be
cleared to ‘0’.
10
NVM All Fast Copy (NVMAFC): If this bit is set to ‘1’, then within this NVM
subsystem, all copy operations performed by the controller are fast copy
operations (refer to the Fast copy operations section of the NVM Command
Set Specification).
If this bit is  cleared to ’0’, then within this NVM subsystem, some copy
operations performed by the controller may not be fast copy operations.
9
NVM Copy Single Atomicity (NVMCSA):  If this bit is set to ‘1’, then the
write portion of any NVM Command Set Copy command is performed as a
single write command to which the atomicity requirements described in the
Atomic operation section of the NVM Command Set Specification apply. If
this bit is cleared to ‘0’, then the atomicity behavior of the write portion of
Copy commands is specified by the Copy atomicity section of the NVM
Command Set Specification.
This bit should be set to ‘1’ if the controller supports the NVM Command Set
Copy command. The Copy atomicity section of the NVM Command Set
Specification specifies additional conditions under which this bit shall be set
to ‘1’.
This bit is ignored by the host if the controller does not support the NVM
Command Set Copy command.
8
Copy Support (NVMCPYS): If this bit is set to ‘1’, then the controller
supports the NVM Command Set Copy command. If this bit is cleared to ‘0’,
then the controller does not support the NVM Command Set Copy
command.
7
Verify Support (NVMVFYS): If this bit is set to ‘1’, then the controller
supports the NVM Command Set Verify command and the Verify Size Limit
(VSL) field indicates the recommended maximum data size for Verify
commands. If this bit is cleared to ‘0’, then controller support of the NVM
Command Set Verify command is indicated by a non -zero data size limit in
the VSL field.
6
Timestamp Support (TSS): If this bit is set to ‘1’, then the controller
supports the Timestamp feature. If this bit is cleared to ‘0’, then the controller
does not support the Timestamp feature. Refer to section 5.1.25.1.7.
