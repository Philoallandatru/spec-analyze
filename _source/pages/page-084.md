---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 84
headings: []
---

# Source page 84

NVM Express® Base Specification, Revision 2.1

62
Figure 42: Offset 1Ch: CSTS – Controller Status
Bits Type Reset1 Description
• if the CC.EN bit is set to ‘1’, after a shutdown operation  then a CLR (e.g., a
Controller Reset)  followed by enabling the controller (i.e., host sets the
CC.EN bit from ‘0’ to ‘1’) is required (refer to section 3.6.1). If a host submits
commands to the controller without a prior CLR, t hen t he behavior is
undefined; and
• if the CC.EN bit is cleared to ‘0’, then:
o a CLR followed by enabling the controller  is required  (refer to
sections 3.6.1 and 3.6.2); or
o the CC.EN bit is required to be set to ‘1’ and the CC.SHN bit is
required to be cleared to 00b with the same write to the CC property
(refer to sections 3.6.1 and 3.6.2).
If the CSTS.ST bit is set to ‘1’ and this field is set to 10b  (i.e., NVM Subsystem
Shutdown processing is reported as complete) , then an NVM Subsystem Reset
followed by enabling the controller is required to start executing commands  (refer to
section 3.6.3). If a host submits commands to the controller without a prior NVM
Subsystem Reset, then the behavior is undefined.
01 RO HwInit
Controller Fatal Status (CFS):  This bit is set to ’1’ when a fatal controller error
occurred that could not be communicated in the appropriate Completion Queue. This
bit is cleared to ‘0’ when a fatal controller error has not occurred. Refer to section 9.5.
The reset value of this bit is set to '1' when a fatal controller error is detected during
controller initialization.
00 RO 0b
Ready (RDY): This bit is set to ‘1’ when the controller is ready to process submission
queue entries after the CC.EN bit is set to ‘1’. This bit shall be cleared to ‘0’ when the
CC.EN bit is cleared to ‘0’ once the controller is ready to be re -enabled. Commands
should not be submitted to the controller until this bit is set to ‘1’ after the CC.EN bit is
set to ‘1’. Failure to follow this recommendation produces undefined results. Refer to
the definition of the CAP.TO field, section 3.5.3, and section 3.5.4 for timing
information related to this field.
If an NVM Subsystem Shutdown that affects this controller is reported as in progress
or is reported as complete (i.e., the CSTS.ST bit is set to ‘1’ and the CSTS.SHST field
is set to 01b or is set to 10b), then an NVM Subsystem Reset is required before this
bit is allowed to be set to ‘1’ from ‘0’. Refer to section 3.6.3.
If a controller shutdown is reported as in progress or is reported as complete (i.e., the
CSTS.ST bit is cleared to ‘0’ and the CSTS.SHST field is set to 01b or is set to 10b),
then before this bit is allowed to be set to ‘1’ from ‘0’, controller shutdown processing
shall stop (e.g., complete or be terminated) and the CSTS.SHST field shall be cleared
to 00b.
Notes:
1. During a Controller Level Reset, the field and bit values may transition to values other than the reset value prior
to indicating the reset value.
