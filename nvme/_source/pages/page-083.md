---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 83
headings: []
---

# Source page 83

NVM Express® Base Specification, Revision 2.1

61
Figure 42: Offset 1Ch: CSTS – Controller Status
Bits Type Reset1 Description
05 RO 0b
Processing Paused (PP): This bit indicates whether the controller is processing
commands. If this bit is cleared to ‘0’, then the controller is processing commands
normally. If this bit is set to ‘1’, then the controller has temporarily stopped processing
commands in order to handle an event (e.g., firmware activation). This bit is only valid
when CC.EN is set to ‘1’ and CSTS.RDY is set to ‘1’.
04 RWC HwInit
NVM Subsystem Reset Occurred (NSSRO): The initial value of this bit is set to '1' if
the last occurrence of an NVM Subsystem Reset  (refer to section 3.7.1) occurred
while power was applied to the domain. The initial value of this bit is cleared to '0'
following an NVM Subsystem Reset due to application of power to the domain. This
bit is only valid if the controller supports the NVM Subsystem Reset feature defined in
section 3.7.1 as indicated by CAP.NSSRS set to ‘1’.
The reset value of this bit is cleared to '0' if an NVM Subsystem Reset causes
activation of a new firmware image in the domain.
03:02 RO 00b
Shutdown Status (SHST): This field indicates the status of shutdown processing that
is initiated by the host setting the CC.SHN field, the host setting the NSSD.NSSC field,
or a Management Endpoint processing an NVMe-MI Shutdown command (refer to the
NVM Express Management Interface Specification). Shutdown processing is able to
occur on this controller as a consequence of a host setting the NSSD.NSSC field on
another controller to initiate an NVM Subsystem Shutdown that affects this controller.
The shutdown status values are defined as:
Value Definition
00b Normal operation (no shutdown has been requested)
01b Shutdown processing in progress
10b Shutdown processing complete
11b Reserved
If this field is set to 01b (i.e., shutdown processing in progress), then:
• an NVM Subsystem Reset aborts both a controller shutdown and an NVM
Subsystem Shutdown; and
• any other type of Controller Level Reset (CLR):
o may or may not abort a controller shutdown; and
o shall not abort an NVM Subsystem Shutdown.
If this field is cleared to 00b (i.e., normal operation) when a CLR is initiated, then that
CLR shall not change the value of this field.
If this field is set to 01b when a CLR is initiated , and shutdown processing is not
aborted by that CLR, then that CLR shall not change the value of this field.
If this field is set to 01b when a CLR is initiated and shutdown processing is aborted
by that CLR, then that CLR shall clear this field to 00b.
If this field is set to 10b (i.e., shutdown processing complete) when a CLR is initiated
by NVM Subsystem Reset, then that CLR shall clear this field to 00b.
If this field is set to 10b when a CLR is initiated by a method other than NVM
Subsystem Reset and:
• the CSTS.ST bit is set to ‘1’, then that CLR shall not change the value of this
field; and
• the CSTS.ST bit is cleared to ‘0’, then that CLR shall clear this field to 00b
If the CSTS.ST bit is cleared to ‘0’ and this field is set to 10b (i.e., controller shutdown
processing is reported as complete) , then to start executing commands on the
controller:
