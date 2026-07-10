---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 511
headings: ["8.1.3.3.3 Interactions between Boot Partition Write Protection Mechanism"]
---

# Source page 511

NVM Express® Base Specification, Revision 2.1

489
is permanent). Once RPMB Boot Partition Write Protection is enabled, Boot Partitions are able to be
modified only after write unlocking the Boot Partition using RPMB.
After enabling RPMB Boot Partition Write Protection:
a) The default state for all Boot Partitions is the Write Locked state. In this state, a host may read a
Boot Partition. In this state, the controller rejects attempts to write to a Boot Partition using the
Firmware Commit command;
b) Each Boot Partition may be locked or unlocked independently using the corresponding bit in the
Device Configuration Block data structure. A Boot Partition may be unlocked in the same command
that enables RPMB Boot Partition Write Protection; and
c) If any Boot Partition has been unlocked, a power cycle or Controller Level Reset event results in
that Boot Partition becoming write locked.
8.1.3.3.3 Interactions between Boot Partition Write Protection Mechanism
Figure 594 shows an overview of the Boot Partition write protection states for each Boot Partition when
both Boot Partition write protection mechanisms are supported. Supporting both Boot Partition write
protection mechanisms is discouraged, as specified in section 8.1.3.3.
