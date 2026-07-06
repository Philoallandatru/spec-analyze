---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 135
headings: ["3.6.3 NVM Subsystem Shutdown"]
---

# Source page 135

NVM Express® Base Specification, Revision 2.1

113
then it is strongly recommended that the host take the steps described in section 9.6 to avoid possible data
corruption caused by interaction between outstanding commands and subsequent commands submitted
by that host to another controller.
The CC.EN field is not used to shutdown the controller (i.e., it is used for Controller Reset).
From the time a controller shutdown is initiated until:
• a Controller Level Reset occurs; or
• the controller, if dynamic, is removed from the NVM subsystem,
the controller shall:
• process only Fabrics commands (refer to Figure 540); and
• disable the Keep Alive timer, if supported.
After the CC.EN bit transitions to ‘0’ (i.e., due to Controller Level Reset), the association between the host
and controller shall be preserved for at least 2 minutes. After this time, the association may be removed if
the controller has not been re-enabled.
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
00b with a single Property Set command (refer to section 6.6) that changes the CC property
(refer to Figure 41). The controller clears the CSTS.SHST field to 00b in response to that
write.
The initialization sequence (refer to section 3.5) should then be executed on that controller.
 NVM Subsystem Shutdown
An NVM Subsystem Shutdown initiates a shutdown of all controllers in a domain or NVM subsystem from
a single controller.
Interactions between NVM Subsystem Shutdown and Power Loss Signaling processing are described in
section 8.2.5.
A controller indicates support for the NVM Subsystem Shutdown Feature by setting the CAP.NSSS bit to
‘1’ (refer to Figure 36).
The NVM Subsystem Shutdown Feature defined in this revision of the NVM Express Base Specification
includes some functionality that differs from the functionality of the NVM Subsystem Shutdown Feature
defined in revision 2.0 of the NVM Express Base Specific ation. A controller indicates support for these
functionality differences by setting the NVM Subsystem Shutdown Enhancements Supported
(CAP.NSSES) bit to ‘1’ (refer to Figure 36).
If a controller sets the CAP.NSSES bit to ‘1’, then while an NVM Subsystem Shutdown is reported as in
progress or is reported as complete (i.e., while the CSTS.ST bit is set to ‘1’ and the CSTS.SHST field is set
to 01b or is set to 10b):
a. a Controller Reset initiates a Controller Level Reset (CLR) (refer to section 3.7.2); and
b. the values of both the CSTS.ST bit and the CSTS.SHST field (refer to Figure 40) are not changed
by a CLR initiated by any method other than an NVM Subsystem Reset.
