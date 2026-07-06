---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 489
headings: ["7.4.2 Command Completion", "7.5 Reservation Acquire command"]
---

# Source page 489

NVM Express® Base Specification, Revision 2.1

467
• prior to processing that I/O Management Send command; or
• upon the completion of that I/O Management Send command.
Figure 567: Reclaim Unit Handle Update – Data Buffer
Bytes Description
Placement Identifier List
01:00 Placement Identifier 1: This field specifies the first Placement Identifier that indicates a Placement
Handle and a Reclaim Group Identifier. Refer to Figure 282 and Figure 283.
03:02 Placement Identifier 2: This field specifies the second Placement Identifier that indicates a
Placement Handle and a Reclaim Group Identifier, if any. Refer to Figure 282 and Figure 283.
…
(NPID*2)+1:
(NPID*2)
Placement Identifier NPID: This field specifies the last Placement Identifier that indicates a
Placement Handle and a Reclaim Group Identifier, if any. Refer to Figure 282 and Figure 283.
If Flexible Data Placement is disabled in the Endurance Group containing the specified namespace, then
the controller shall abort the command with a status code of FDP Disabled.
 Command Completion
When the command is completed with success or failure, the controller shall post a completion queue entry
to the associated I/O Completion Queue indicating the status for the command.
7.5 Reservation Acquire command
The Reservation Acquire command is used to:
• acquire a reservation on a namespace;
• preempt a reservation held on a namespace; or
• preempt a reservation held on a namespace and abort outstanding commands for that namespace.
The command uses Command Dword 10 and a Reservation Acquire data structure in memory. If the
command uses PRPs for the data transfer, then PRP Entry 1 and PRP Entry 2 fields are used. If the
command uses SGLs for the data transfer, then the SGL Entry 1 fie ld is used. All other command specific
fields are reserved.
Figure 568: Reservation Acquire – Data Pointer
Bits Description
127:00 Data Pointer (DPTR):  This field specifies the  location of a data buffer where data is transferred from .
Refer to Figure 92 for the definition of this field.

Figure 569: Reservation Acquire – Command Dword 10
Bits Description
31:16 Reserved
15:08 Reservation Type (RTYPE): This field specifies the type of reservation to be created. The field is defined
in Figure 571.
07:05 Reserved
