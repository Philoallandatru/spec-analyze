---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 76
headings: []
---

# Source page 76

NVM Express® Base Specification, Revision 2.1

54
Figure 36: Offset 0h: CAP – Controller Capabilities
Bits Type Reset Description
35:32 RO Impl
Spec
Doorbell Stride (DSTRD):  Each Submission Queue and Completion Queue
Doorbell property is 32 -bits in size. This property indicates the stride between
doorbell properties. The stride is specified as (2 ^ (2 + DSTRD)) in bytes. A value
of 0h indicates a stride of 4 bytes, where the doorbell properties are packed without
reserved space between each property. Refer to section 8.2.2.
For NVMe over Fabrics  I/O controllers, this property shall be cleared to a fixed
value of 0h.
31:24 RO Impl
Spec
Timeout (TO): This is the worst-case time that host software should wait for the
CSTS.RDY bit to transition from:
a) ‘0’ to ‘1’ after the CC.EN bit transitions from ‘0’ to ‘1’; or
b) ‘1’ to ‘0’ after the CC.EN bit transitions from ‘1’ to ‘0’.
This worst-case time may be experienced after events such as an abrupt shutdown,
loss of main power without shutting down the controller , or activation of a new
firmware image; typical times are expected to be much shorter.
This field is in 500 millisecond units. The maximum value of this field is FFh, which
indicates a 127.5 second timeout.
If the Controller Ready Independent of Media Enable (CC.CRIME) bit is cleared to
‘0’ and the worst-case time for the CSTS.RDY bit to change state is due to enabling
the controller after the CC.EN bit transitions from ‘0’ to ‘1’, then this field shall be
set to:
a) the value in  the Controller Ready With Media Timeout (CRTO.CRWMT)
field; or
b) FFh if the value in the CRTO.CRWMT field is greater than FFh.
If the Controller Ready Independent of Media Enable (CC.CRIME) bit is set to ‘1’
and the worst-case time for the CSTS.RDY bit to change state is due to enabling
the controller after the CC.EN bit transitions from ‘0’ to ‘1’, then this field shall be
set to:
a) the value in the Controller Ready Independent of Media Timeout
(CRTO.CRIMT); or
b) FFh if the value in the CRTO.CRIMT field is greater than FFh.
Controllers that support the CRTO register (refer to Figure 57) are able to indicate
larger timeouts for enabling the controller. Host software should use the value in
the CRTO.CRWMT field or the CRTO.CRIMT field depending on the controller
ready mode indicated by the CC.CRIME bit to determine the worst -case timeout
for the CSTS.RDY bit to transition from ‘0’ to ‘1’ after the CC.EN bit transitions from
‘0’ to ‘1’. Host software that is based on  revisions earlier than NVM Express Base
Specification, Revision 2.0 is not required to wait for more than 127.5 seconds for
the CSTS.RDY bit to transition.
Refer to sections 3.5.3 and 3.5.4 for more information.
23:19 RO 0h Reserved
