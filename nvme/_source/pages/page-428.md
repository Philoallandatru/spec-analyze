---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 428
headings: ["5.1.25.2.2 Interrupt Coalescing (Feature Identifier 08h)"]
---

# Source page 428

NVM Express® Base Specification, Revision 2.1

406
Figure 443: Number of Queues – Command Dword 11
Bits Description
15:00
Number of I/O Submission Queues Requested (NSQR): Indicates the number of I/O Submission
Queues requested by software. This number does not include the Admin Submission Queue. A minimum
of one queue shall be requested, reflecting that the minimum support is for one I/O Submission Queue.
This is a 0’s base d value. The maximum value that may be specified is 65,534 (i.e., 65,535 I/O
Submission Queues). If the value specified is 65,535, the controller should abort the command with a
status code of Invalid Field in Command.
Note: The value allocated may be smaller or larger than the number of queues requested, often in virtualized
implementations. The controller may not have as many queues to allocate as are requested. Alternatively,
the controller may have an allocation unit of queues (e.g., power of two) and may supply more queues to
host software to satisfy its allocation unit.
Figure 444: Number of Queues – Completion Queue Entry Dword 0
Bits Description
31:16
Number of I/O Completion Queues Allocated (NCQA): Indicates the number of I/O Completion
Queues allocated by the controller. A minimum of one queue shall be allocated, reflecting that the
minimum support is for one I/O Completion Queue. The value may not match the number requested by
host software. This is a 0’s based value.
15:00
Number of I/O Submission Queues Allocated (NSQA): Indicates the number of I/O Submission
Queues allocated by the controller. A minimum of one queue shall be allocated, reflecting that the
minimum support is for one I/O Submission Queue. The value may not match the number requested by
host software. This is a 0’s based value.
5.1.25.2.2 Interrupt Coalescing (Feature Identifier 08h)
This Feature configures controller configures interrupt coalescing settings. The controller should signal an
interrupt when either the Aggregation Time or the Aggregation Threshold conditions are met. If either the
Aggregation Time or the Aggregation Threshold fields are cleared to 0h, then an interrupt may be generated
(i.e., interrupt coalescing is implicitly disabled). This Feature app lies only to the I/O Queues. It is
recommended that interrupts for commands that complete in error are not coalesced. The settings are
specified in Command Dword 11.
If the controller detects that interrupts are already being processed for this vector, then the controller may
delay additional interrupts. Specifically, if the Completion Queue Head Doorbell property that is associated
with a particular interrupt vector is being updated, then  the controller has a positive indication that
completion queue entries are already being processed. In this case, the aggregation time and/or the
aggregation threshold may be reset/restarted upon the associated property write. This ma y result in
interrupts being delayed indefinitely in certain workloads where the aggregation time or aggregation
threshold is non-zero.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 445 are returned
in Dword 0 of the completion queue entry for that command.
This Feature is valid when the controller is configured for Pin Based, MSI, Multiple MSI or MSI-X interrupts.
There is no requirement for the controller to persist these settings if interrupt modes are changed. It is
recommended that the host re-issue this Feature after changing interrupt modes.
Figure 445: Interrupt Coalescing – Command Dword 11
Bits Description
31:16 Reserved
15:08
Aggregation Time (TIME): Specifies the recommended maximum time in 100 microsecond increments
that a controller may delay an interrupt due to interrupt coalescing. A value of 0h corresponds to no
delay. The controller may apply this time per interrupt vector or across all interrup t vectors. The reset
value of this setting is 0h.
