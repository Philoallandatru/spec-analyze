---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 446
headings: ["5.2.5.1 Command Completion", "5.2.6 Virtualization Management command"]
---

# Source page 446

NVM Express® Base Specification, Revision 2.1

424
Figure 486: Doorbell Buffer Config – Shadow Doorbell and EventIdx
Start
(Offset in Buffer)1, 2
End
(Offset in Buffer)1, 2 Description 2
1. The offsets in Start and End are referenced to the value provided in PRP1 for the doorbell buffer and to the
value provided in PRP2 for the EventIdx buffer.
2. The value of y is equal to max(NSQA, NCQA).

Figure 487: Doorbell Buffer Config – PRP Entry 1
Bits Description
63:00
PRP Entry 1 (PRP1): This field specifies a 64-bit base memory address pointer to the Shadow Doorbell
buffer with the definition specified in Figure 486. The Shadow Doorbell buffer is updated by the host. This
buffer shall be memory page aligned.

Figure 488: Doorbell Buffer Config – PRP Entry 2
Bits Description
63:00
PRP Entry 2 (PRP2): This field specifies a 64 -bit base memory address pointer to the EventIdx buffer
with the definition specified in Figure 486. The EventIdx buffer is updated by the para -virtualized
controller. This buffer shall be memory page aligned.
 Command Completion
When the command is completed, the controller posts a completion queue entry to the Admin Completion
Queue indicating the status for the command. If the Shadow Doorbell buffer or EventIdx buffer memory
addresses are invalid, then a status code of Invalid Field in Command shall be returned.
 Virtualization Management command
The Virtualization Management command is supported by primary controllers that support the Virtualization
Enhancements capability. This command is used for several functions:
• Modifying Flexible Resource allocation for the primary controller;
• Assigning Flexible Resources for secondary controllers; and
• Setting the Online and Offline state for secondary controllers.
Refer to section 8.2.6 for more on the Virtualization Enhancements capability and the Virtualization
Management command.
The Virtualization Management command uses the Command Dword 10 and Command Dword 11 fields.
All other command specific fields are reserved.
If the action requested specifies a range of controller resources that:
a) does not exist;
b) is a Private Resource (e.g., VQ resources are requested when VQ resources are not supported, VI
resources are requested when VI resources are not supported); or
c) is currently in use (e.g., the number of Controller Resources (NR) is greater than the number of
remaining available flexible resources),
then the command is aborted with a status code of Invalid Resource Identifier.
Figure 489: Virtualization Management – Command Dword 10
Bits Description
31:16 Controller Identifier (CNTLID): This field indicates the controller for which controller resources are to be
modified.
15:11 Reserved
