---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 508
headings: ["8.1.3.3.1 Set Features Boot Partition Write Protection"]
---

# Source page 508

NVM Express® Base Specification, Revision 2.1

486
Controller data structure (refer to Figure 312). Only controllers compliant with NVM Express Base
Specification, Revision 2.0 and earlier are allowed to report a value of 00b (i.e., support not specified) in
that field. If a controller reports the 00b value, a host is able to determine if RPMB Boot Pa rtition Write
Protection is supported by checking whether the controller supports both Boot Partitions (refer to section
3.1.4.1) and RPMB (refer to section 5.1.13.2.1) because a controller compliant with NVM Express Base
Specification, Revision 2.0 and earlier is required to support RPMB Boot Partition Write Protection when
the controller supports both Boot Partitions and RPMB.
If Set Features Boot Partition Write Protection is supported and either:
1) RPMB Boot Partition Write Protection is not supported; or
2) RPMB Boot Partition Write Protection is not enabled,
then the Boot Partition write protection states are able to be configured via the Boot Partition Write
Protection Config feature. Section 8.1.3.3.1 covers the case when only the Boot Partition Write Protection
Config feature is supported.
If RPMB Boot Partition Write Protection is supported and enabled, then the Boot Partition write protection
states are able to be configured using RPMB (refer to section 8.1.21). Section 8.1.3.3.2 covers the case
where only RPMB Boot Partition Write Protection is supported.
Only one mechanism controls the Boot Partition write protection states at a time. Section 8.1.3.3.3 covers
considerations when both Boot Partition write protection mechanisms are supported.
If any Boot Partition is shared across multiple controllers (refer to section 8.1.3), then the write protection
state of the Boot Partition shall be enforced by all controllers that share that Boot Partition.
8.1.3.3.1 Set Features Boot Partition Write Protection
Figure 590 shows an overview of the Boot Partition write protection states for each Boot Partition when only
Set Features Boot Partition Write Protection is supported.
Figure 590: Set Features Boot Partition Write Protection State Machine Model
Write Locked Write Unlocked
Write Locked
Set Features
Set Features
Set Features
power up
or
power cycle

Figure 591 defines the write protection states per Boot Partition if Set Features Boot Partition Write
Protection is supported.
