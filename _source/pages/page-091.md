---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 91
headings: ["3.1.4.20 Offset 64h: NSSD – NVM Subsystem Shutdown", "3.1.4.21 Offset 68h: CRTO – Controller Ready Timeouts"]
---

# Source page 91

NVM Express® Base Specification, Revision 2.1

69
7:4 RO 0h Reserved
3:0 RO Impl
Spec
CMB Sustained Write Throughput Units (CMBSWTU): Indicates the granularity of the
CMB Sustained Write Throughput field.
Value Granularity
0h Bytes/second
1h 1 KiB/second
2h 1 MiB/second
3h 1 GiB/second
4h – Fh Reserved

 Offset 64h: NSSD – NVM Subsystem Shutdown
This optional property provides host software with the capability to initiate a normal or an abrupt NVM
Subsystem Shutdown.
Support for this property is indicated by the state of the NVM Subsystem Shutdown Supported (CAP.NSSS)
field. If the property is not supported, then the address range occupied by the register is reserved.
The NVM Subsystem Shutdown Enhancements Supported (CAP.NSSES) bit affects the functionality
invoked by host modification of this property (refer to section 3.6.3).
Figure 56: Offset 64h: NSSD – NVM Subsystem Shutdown
Bits Type Reset Description
31:00 RW 0h
NVM Subsystem Shutdown Control (NSSC):  A write of the value 4E726D6Ch
("Nrml") to this field initiates a normal NVM Subsystem Shutdown on every controller:
• in the domain associated with the controller when CAP.CPS is set to 10b (i.e.,
domain scope) as specified in section 3.6.3.2 or
• in the NVM subsystem when CAP.CPS is set to 11b (i.e., NVM subsystem
scope) in the NVM subsystem as specified in section 3.6.3.1.
A write of the value 41627074h ("Abpt") to this field initiates an abrupt NVM subsystem
shutdown on every controller:
• in the domain associated with the controller when CAP.CPS is set to 10b as
specified in section 3.6.3.2; or
• in the NVM subsystem when CAP.CPS is set to 11b in the NVM subsystem
as specified in section 3.6.3.1.
A write of any other value to this field has no functional effect on the operation of the
NVM subsystem. This field shall return the value 0h when read.
 Offset 68h: CRTO – Controller Ready Timeouts
This property indicates the controller ready timeout values. This property is mandatory for controllers
compliant with NVM Express Base Specification revision 2.0 and later.
