---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 134
headings: ["3.6.2 Message-based Controller Shutdown (Fabrics)"]
---

# Source page 134

NVM Express® Base Specification, Revision 2.1

112
C. an Admin command that requires access to the media (refer to Figure 84) and specifies the Ignore
Shutdown bit set to ‘1’ is processed by the controller via the out -of-band mechanism (refer to the
NVM Express Management Interface Specification).
If a Controller Level Reset occurs while controller shutdown processing is reported as complete (i.e., the
CSTS.ST bit is cleared to ‘0’ and the CSTS.SHST field is set to 10b), then the controller may remain ready
to be powered off (e.g., the media remains in the shutdown state) or the controller may cease being ready
to be powered off (e.g., because the controller is preparing the media for use (refer to section 3.7.2)).
If the power scope for the controller includes multiple controllers (e.g., the CAP.CPS field is set to 10b or is
set to 11b), and any controller included in that power scope is not ready to be powered off, then the portion
of the NVM subsystem included in that power scope is not ready to be powered off.
To start executing commands on the controller after that controller reports controller shutdown processing
complete (i.e., the CSTS.ST bit is cleared to ‘0’ and the CSTS.SHST field is set to 10b) utilizing the CC.EN
bit:
• if the CC.EN bit is set to ‘1’, then a Controller Level Reset is required to clear the CC.EN bit to ‘0’
on that controller and the CC.EN bit is subsequently required to be set to ‘1’ as part of the
initialization sequence (refer to section 3.5); and
• if the CC.EN bit is cleared to ‘0’, then:
o a Controller Level Reset is required and the CC.EN bit is subsequently required to be set
to ‘1’ as part of the initialization sequence (refer to section 3.5); or
o the CC.EN bit is required to be set to ‘1’ and the CC.SHN field is required to be cleared to
00b with the same write to the CC property (refer to  Figure 41). The controller clears the
CSTS.SHST field to 00b in response to that write.
The initialization sequence (refer to section 3.5) should then be executed on that controller.
It is an implementation choice whether the host aborts all outstanding commands to the Admin Queue prior
to the controller shutdown. The only commands that should be outstanding to the Admin Queue when the
controller reports shutdown processing complete are Asynchronous Event Request commands.
If the host is no longer able to communicate with the controller before that host receives either :
• completions for all outstanding commands submitted to that controller (refer to section 3.4.5); or
• a CSTS.SHST field value that indicates that the controller shutdown is complete,
then it is strongly recommended that the host take the steps described in section 9.6 to avoid possible data
corruption caused by interaction between outstanding commands and subsequent commands submitted
by that host to another controller.
 Message-based Controller Shutdown (Fabrics)
To initiate a shutdown of a controller, the host should use the Property Set command (refer to section 6.6)
to set the Shutdown Notification (CC.SHN) field to:
• 01b to initiate a normal shutdown operation; or
• 10b to initiate an abrupt shutdown.
After the host initiates a controller shutdown, the host may either disconnect at the NVMe Transport level
or the host may choose to poll CSTS.SHST to determine when the controller shutdown is complete (i.e.,
the controller should not initiate a disconnect at the NVMe Transport level). It is an implementation choice
whether the host aborts all outstanding commands prior to initiating the shutdown.
If the host is no longer able to communicate with the controller before that host receives either :
• completions for all outstanding commands submitted to that controller (refer to section 3.4.5); or
• a CSTS.SHST field value that indicates that the controller shutdown is complete,
