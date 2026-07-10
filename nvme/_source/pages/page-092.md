---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 92
headings: ["3.1.4.22 Offset E00h: PMRCAP – Persistent Memory Region Capabilities"]
---

# Source page 92

NVM Express® Base Specification, Revision 2.1

70
Figure 57: Offset 68h: CRTO – Controller Ready Timeouts
Bits Type Reset Description
31:16 RO Impl
Spec
Controller Ready Independent of Media Timeout (CRIMT):  If the
CAP.CRMS.CRIMS bit is cleared to ‘0’, then the controller shall clear this field to
0h and the host should ignore this field.
If the CAP.CRMS.CRIMS bit is set to ‘1’, then this field contains the worst -case
time that host software should wait after CC.EN transitions from ‘0’ to ‘1’ for the
controller to become ready and be able to successfully process all commands
that do not acc ess attached namespaces and Admin commands that do not
require access to media when the controller is in Controller Ready Independent
of Media mode (i.e., the CC.CRIME bit is set to ‘1’). Attached namespaces and
media required to process Admin commands may or may not be ready within this
time period (refer to section 3.5.3, section 3.5.4, and Figure 84).
This worst -case time may be experienced after events such as an abrupt
shutdown or activation of a new firmware image; typical times are expected to be
much shorter. This field is in 500 millisecond units.
The value of this field should not exceed FFh (i.e., 127.5 seconds).
15:0 RO Impl
Spec
Controller Ready With Media Timeout (CRWMT): This field contains the worst-
case time that host software should wait after CC.EN transitions from ‘0’ to ‘1’
for:
a) the controller to become ready and be able to successfully process all
commands; and
b) all attached namespaces and media required to process Admin
commands to become ready,
independent of which ready mode (refer to CC.CRIME) the controller is in (refer
to section 3.5.3 and section 3.5.4).
This worst -case time may be experienced after events such as an abrupt
shutdown or activation of a new firmware image; typical times are expected to be
much shorter. This field is in 500 millisecond units.
The value of this field shall be greater than or equal to the value of the
CRTO.CRIMT field and may be significantly larger than the value of the
CRTO.CRIMT field.
 Offset E00h: PMRCAP – Persistent Memory Region Capabilities
This property indicates capabilities of the Persistent Memory Region. If the controller does not support the
Persistent Memory Region feature, then this property shall be cleared to 0h.
This property shall not be reset by Controller Reset.
Figure 58: Offset E00h: PMRCAP – Persistent Memory Region Capabilities
Bits Type Reset Description
31:25 RO 0h Reserved
24 RO Impl
Spec
Controller Memory Space Supported (CMSS): If this bit is set to ‘1’, then the addresses
supplied by the host are permitted to reference the Persistent Memory Region only if the
host has enabled the Persistent Memory Region’s controller memory space.
If the controller supports referencing the Persistent Memory Region with host -supplied
addresses, then this bit shall bet set to ‘1’. Otherwise, this bit shall be cleared to ‘0’.
23:16 RO Impl
Spec
Persistent Memory Region Timeout (PMRTO): This field contains the minimum
amount of time that host software should wait for the Persistent Memory Region to
become ready or not ready after PMRCTL.EN is modified. The time in this field is
expressed in Persistent Memory Region time units (refer to PMRCAP.PMRTU).
15:14 RO 00b Reserved
