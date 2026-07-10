---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 429
headings: ["5.1.25.2.3 Interrupt Vector Configuration (Feature Identifier 09h)", "5.1.25.2.4 Host Memory Buffer (Feature Identifier 0Dh)"]
---

# Source page 429

NVM Express® Base Specification, Revision 2.1

407
Figure 445: Interrupt Coalescing – Command Dword 11
Bits Description
07:00
Aggregation Threshold (THR): Specifies the recommended minimum number of completion queue
entries to aggregate per interrupt vector before signaling an interrupt to the host. This is a 0’s based
value. The reset value of this setting is 0h.
5.1.25.2.3 Interrupt Vector Configuration (Feature Identifier 09h)
This Feature configures settings specific to a particular controller interrupt vector. The settings are specified
in Command Dword 11.
By default, coalescing settings are enabled for each interrupt vector. Interrupt coalescing is not supported
for the Admin Completion Queue.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 446 are returned
in Dword 0 of the completion queue entry for that command for the Interrupt Vector specified in Command
Dword 11.
Prior to issuing a Set Features command that specifies this Feature, the host shall configure the specified
Interrupt Vector with an I/O Completion Queue (refer to section 5.2.1). If the specified Interrupt Vector is
invalid, or not associated with an existing I/O Completion Queue (refer to Figure 475), then the controller
should abort the command with a status code of Invalid Field in Command.
Figure 446: Interrupt Vector Configuration – Command Dword 11
Bits Description
31:17 Reserved
16
Coalescing Disable (CD): If this bit is set to ‘1’, then any interrupt coalescing settings shall not be applied
for this interrupt vector. If this bit is cleared to ‘0’, then interrupt coalescing settings apply for this interrupt
vector.
15:00 Interrupt Vector (IV): This field specifies the interrupt vector for which the configuration settings are
applied.
5.1.25.2.4 Host Memory Buffer (Feature Identifier 0Dh)
This Feature controls use of the Host Memory Buffer by the controller. The attributes are specified in
Command Dword 11, Command Dword 12, Command Dword 13, Command Dword 14, and Command
Dword 15.
The Host Memory Buffer feature provides a mechanism for the host to allocate a portion of host memory
for the exclusive use of the controller. After a successful completion of a Set Features command enabling
the host memory buffer, the host shall not write to:
a) The Host Memory Descriptor List (refer to Figure 452); and
b) the associated host memory region (i.e., the memory regions described by the Host Memory
Descriptor List),
until the host memory buffer has been disabled.
If the host memory buffer is enabled, then a Set Features command to enable the host memory buffer (i.e.,
the EHM bit (refer to Figure 447) set to ‘1’) shall abort with a status code of Command Sequence Error.
If the host memory buffer is not enabled, then a Set Features command to disable the host memory buffer
(i.e., the EHM bit (refer to Figure 447) cleared to ‘0’) shall succeed without taking any action.
After a successful completion of a Set Features command that disables the host memory buffer, the
controller shall not access any data in the host memory buffer until the host memory buffer has been
enabled. The controller should retrieve any necessary dat a from the host memory buffer in use before
posting the completion queue entry for the Set Features command that disables the host memory buffer.
Posting of the completion queue entry for the Set Features command that disables the host memory buffer
