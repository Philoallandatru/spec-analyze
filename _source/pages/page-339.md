---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 339
headings: []
---

# Source page 339

NVM Express® Base Specification, Revision 2.1

317
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
358 O P P
Key Per I/O Capabilities (KPIOC): This field indicates the attributes for the Key Per I/O
capability (refer to section 8.1.11).
Bits Description
7:2 Reserved
1
Key Per I/O Scope (KPIOSC):  If this bit is set to ‘1’, then the Key Per I/O
capability applies to all namespaces in the NVM subsystem when the Key Per
I/O capability is enabled. If  this bit is  cleared to ‘0’, then the Key Per I/O
capability does not apply to all namespaces in the NVM subsystem and is
allowed to be independently enabled and disabled uniquely on each
namespace within the NVM subsystem. If the KPIOS bit is cleared to ‘0’, then
this bit should be ignored by the host.
0
Key Per I/O Supported (KPIOS):  If this bit is set to ‘1’, then the controller
supports the Key Per I/O capability. If this bit is cleared to ‘0’, then the
controller does not support the Key Per I/O capability.

359    Reserved
361:360 O O R
Maximum Processing Time for Firmware Activation With out Reset (MPTFAWR):
This field shall indicate the estimated maximum time in 100 ms units required by the
controller to process a Firmware Commit command that specifies a value of 011b in the
Commit Action field (i.e., firmware activation without reset).
This field applies to Firmware Commit commands received on an NVM Express controller
Admin Submission Queue or received out -of-band on a Management Endpoint (refer to
the NVM Express Management Interface Specification).
If firmware activation without reset is not supported, then this field shall be cleared to 0h.
A value of 0h indicates that no time is indicated. This field shall not be cleared to 0h on
implementations that support firmware activation without reset and that are compliant with
revision 2.1 or later of this specification.
367:362    Reserved
383:368 O R R
Max Endurance Group Capacity (MEGCAP): This field indicates the maximum capacity
of a single Endurance Group. If this field is cleared to 0h, then the NVM subsystem does
not report a maximum Endurance Group Capacity value.
384 O O O
Temperature Threshold Hysteresis Attributes (TMPTHHA): This field indicates
attributes related to temperature threshold hysteresis. Refer to section 5.1.25.1.3.1.
Bits Description
7:3 Reserved
2:0
Temperature Threshold Maximum Hysteresis (TMPTHMH):  This field
indicates the maximum temperature hysteresis value in Kelvins that is
supported by the controller. This is the absolute value of the difference.
If the controller does not support this attribute, this field shall be cleared to 000b.

385    Reserved
387:386 M M M
Command Quiesce Time (CQT): This field indicates the expected worst-case time in 1
millisecond units for the controller to quiesce all outstanding commands (i.e., the
controller shall satisfy all Immediate Abort requirements for those commands (refer to
section 5.1.1.1)) after a Keep Alive Timeout (refer to section 3.9) or other communication
loss (refer to section 9.6). If this field is cleared to 0h, then a command quiesce time is
not reported. If the controller does not require any time to quiesce, the controller should
set this field to 1h (i.e., 1 millisecond).
511:388    Reserved
