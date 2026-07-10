---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 327
headings: []
---

# Source page 327

NVM Express® Base Specification, Revision 2.1

305
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
102 M R R
Boot Partition Capabilities (BPCAP): This field indicates the Boot Partition capabilities
supported by the controller.
Bits Description
07:03 Reserved
02
Set Features Boot Partition Write Protection Support  (SFBPWPS): This
bit indicates if Set Features Boot Partition Write Protection is supported by the
controller. If supported, this capability allows a host to configure Boot Partition
write protection states via the Boot Partition Write Protection Config feature in
the Set Features command. Refer to section 8.1.3.3.1.
Value Definition
0b Set Features Boot Partition Write Protection is not supported
by this controller.
1b Set Features Boot Partition Write Protection is supported by
this controller.

01:00
RPMB Boot Partition Write Protection Support (RPMBBPWPS): This field
indicates if RPMB Boot Partition Write Protection is supported by the
controller. If supported, this capability allows a host to configure Boot Partition
write protection states via RPMB. Refer to section 8.1.3.3.2.
Value Definition
00b
Support for RPMB Boot Partition Write Protection is not
specified. Only controllers compliant with NVM Express
Base Specification, Revision 2.0 and earlier are allowed to
report this value. Refer to section 8.1.3.3 for more details
on when a controller may return this value.
01b RPMB Boot Partition Write Protection is not supported by
this controller.
10b RPMB Boot Partition Write Protection is supported by this
controller.
11b Reserved


103    Reserved
107:104 O O R
NVM Subsystem Shutdown Latency (NSSL): This field indicates the typical latency in
microseconds for an NVM Subsystem Shutdown to complete. Refer to section 3.6.3. A
value of 0h indicates that NVM Subsystem Shutdown Latency is not reported.
109:108    Reserved
110 O O R
Power Loss Signaling Information (PLSI): This field indicates information about Power
Loss Signaling processing capabilities.
Bits Description
7:2 Reserved
1
PLS Forced Quiescence (PLSFQ): If this bit is set to ‘1’, then the controller
supports Power Loss Signaling with Forced Quiescence (refer to section
8.2.5.2). If this bit is cleared to ‘0’, then the controller does not support Power
Loss Signaling with Forced Quiescence.
0
PLS Emergency Power Fail (PLSEPF) : If this bit is set to ‘1’, then the
controller supports Power Loss Signaling with Emergency Power Fail (refer
to section 8.2.5.3). If this bit is cleared to ‘0’, then the controller does not
support Power Loss Signaling with Emergency Power Fail.
