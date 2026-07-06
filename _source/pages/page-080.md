---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 80
headings: []
---

# Source page 80

NVM Express® Base Specification, Revision 2.1

58
Figure 41: Offset 14h: CC – Controller Configuration
Bits Type Reset Description
15:14 RW 00b
Shutdown Notification (SHN):  This field is used to initiate a controller
shutdown when a power down condition is expected. For a normal controller
shutdown, it is expected that the controller is given time to process the controller
shutdown. For an abrupt shutdown, the host may not wait for the controller
shutdown to complete before power is lost.
The controller shutdown notification values are defined as:
Value Definition
00b No notification and no effect
01b Normal shutdown notification
10b Abrupt shutdown notification
11b Reserved
This field should be written by host software prior to any power down condition
and prior to any change of the PCI power management state. It is recommended
that this field also be written prior to a warm  reset (refer to the PCI Express
Base Specification). To determine when the controller shutdown processing is
complete, refer to  the definition of the  CSTS.ST bit and the definition of the
CSTS.SHST field. Refer to section  3.6 for additional shutdown processing
details.
Other fields in this property (including the EN bit) may be modified as part of
updating this field to 01b or 10b to initiate a controller shutdown. If the EN bit is
cleared to ‘0’ such that the EN bit transitions from ‘1’ to ‘0’, then both a Controller
Reset and a controller shutdown occur.
If an NVM Subsystem Shutdown is reported as in progress or is reported as
complete (i.e., CSTS.ST is set to ‘1’, and CSTS.SHST is set to 01b or 10b),
then writes to this field modify the field value but have no effect. Refer to section
3.6.3 for details.
13:11 RW 000b
Arbitration Mechanism Selected (AMS): This field selects the arbitration
mechanism to be used. This value shall only be changed when CC.EN is
cleared to ‘0’. Host software shall only set this field to supported arbitration
mechanisms indicated in CAP.AMS. If this field is set to an unsupported value,
then the behavior is undefined.
For Discovery controllers, this value shall be cleared to 0h.
Value Definition
000b Round Robin
001b Weighted Round Robin with Urgent Priority Class
010b to 110b Reserved
111b Vendor Specific

10:07 RW 0h
Memory Page Size (MPS): This field indicates the host memory page size. The
memory page size is (2 ^ (12 + MPS)). Thus, the minimum host memory page
size is 4 KiB and the maximum host memory page size is 128  MiB. The value
set by host software shall be a supported value as indica ted by the
CAP.MPSMAX and CAP.MPSMIN fields. This field describes the value used for
PRP entry size. This field shall only be modified when CC.EN is cleared to ‘0’.
For Discovery controllers this property shall be cleared to 0h.
