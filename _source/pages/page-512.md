---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 512
headings: []
---

# Source page 512

NVM Express® Base Specification, Revision 2.1

490
Figure 594: Boot Partition Write Protection State Machine Model
      RPMB BP Write Protection Disabled
Write Locked Write Unlocked
Write Locked Until
Power Cycle
Set Features
Set Features
Set Features
power up
or
power cycle
      RPMB BP Write Protection Enabled
Write Locked Write Unlocked
Enable RPMB Boot
Partition Write Protection
Authenticated Device
Configuration Block Write
power up
or
any reset

If both Boot Partition write protection mechanisms are supported by the controller, only one Boot Partition
write protection mechanism controls the write protection states of the Boot Partitions for the controller at
any time. If RPMB Boot Partition Write Protection is enabled, then RPMB Boot Partition Write Protection
controls the Boot Partition write protection states (refer to section 8.1.3.3.2 and section 8.1.21). If RPMB
Boot Partition Write Protection is disabled, then Set Features Boot Partition Write Protection controls the
Boot Partition write protection states (refer to section 8.1.3.3.1 and section 5.1.25.1.32). Control of the Boot
Partition write protection states transitions from Set Features Boot Partition Write Protection to RPMB Boot
Partition Write Protection by enabling RPMB Boot Partition Write Protection when the Boot Partitions are
in either a Write Unlocked or Write Locked state.
If both Boot Partition write protection capabilities are supported and an RPMB authentication key has not
been programmed for RPMB target 0, there is a possibility of malicious bypass of a Boot Partition’s Write
Locked Until Power Cycle state. In order to prevent malicious bypass of a Boot Partition's Write Locked
Until Power Cycle state, a controller that supports both Boot Partition write protection mechanisms is
required to prevent host attempts to enable RPMB Boot Partition Write Protection when either Boot Partition
