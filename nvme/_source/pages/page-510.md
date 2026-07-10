---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 510
headings: []
---

# Source page 510

NVM Express® Base Specification, Revision 2.1

488
Figure 592: RPMB Boot Partition Write Protection State Machine Model
     RPMB BP Write Protection Disabled
     RPMB BP Write Protection Enabled
Write Locked
Write Unlocked
Write Unlocked
power up
or
any reset
Authenticated Device
Configuration Block Write
Enable RPMB Boot
Partition Write Protection
power up
or
any reset

Figure 594 defines the write protection states per Boot Partition if RPMB Boot Partition Write Protection is
supported.
Figure 593: RPMB Boot Partition Write Protection State Definitions
State Definition
Persistent Across
Power Cycles Controller
Level Resets
RPMB Boot Partition Write Protection Disabled
Write Unlocked The Boot Partition is not write locked. Yes Yes
RPMB Boot Partition Write Protection Enabled
Write Unlocked The Boot Partition is not write locked. No No
Write Locked The Boot Partition is write locked. Yes Yes
If Set Features Boot Partition Write Protection is not supported by the controller, then prior to enabling
RPMB Boot Partition Write Protection, the default state for all Boot Partitions is the Write Unlocked state. If
Set Features Boot Partition Write Protection is also supported by the controller, then the default sta te for
all Boot Partitions is the Write Locked state, regardless of whether RPMB Boot Partition Write Protection
has been enabled or not. Refer to section 8.1.3.3.1 for more details on Boot Partition write protection when
RPMB Boot Partition Write Protection is disabled and section 8.1.3.3.3 for additional considerations when
both Boot Partition write protection capabilities are supported.
If Set Features Boot Partition Write Protection is not supported by the controller, then all Boot Partitions
remain unlocked until RPMB Boot Partition Write Protection is enabled by the host. A host enables RPMB
Boot Partition Write Protection by setting the Boot Partition Write Protection Enabled bit in the RPMB Device
Configuration Block data structure (refer to section 8.1.21). Once RPMB Boot Partition Write Protection is
enabled, the controller shall reject Authenticated Device Configuration Block Writes that attempt to disable
the RPMB Boot Partition Write Protection mechanism (i.e., enabling RPMB Boot Partition Write Protection
