---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 82
headings: ["3.1.4.6 Offset 1Ch: CSTS – Controller Status"]
---

# Source page 82

NVM Express® Base Specification, Revision 2.1

60
Figure 41: Offset 14h: CC – Controller Configuration
Bits Type Reset Description
00 RW 0b
Enable (EN): When set to ‘1’, then the controller shall process commands.
When cleared to ‘0’, then the controller shall not process commands nor post
completion queue entries to Completion Queues. When  the host modifies CC
to clear this bit from ‘1’ to ‘0’, the controller is reset (i.e., a Controller Reset, refer
to section 3.7.2). That reset deletes all I/O Submission Queues and I/O
Completion Queues, resets the Admin Submission Queue and the Admin
Completion Queue, and brings the hardware to an idle state. That reset does
not affect transport specific state (e.g., PCI Express registers including MMIO
MSI-X registers), nor the Admin Queue properties (AQA, ASQ, or ACQ). Refer
to section 3.7.2 for the effects of that reset on a ll controller properties. Internal
controller state (e.g., Feature values defined in section 5.1.25 that are not
persistent across power states) are reset to their default values. The controller
shall ensure that there is no impact (e.g., data loss) caused by that Controller
Reset to the results of commands that have had corresponding completion
queue entries posted to an I/O Completion Queue prior to that Controller Reset.
When this bit is cleared to ‘0’, the CSTS.RDY bit is cleared to ‘0’ by the controller
once the controller is ready to be re -enabled. When this bit is set to ‘1’, the
controller sets the CSTS.RDY bit to ‘1’ when it is ready to process commands.
The CSTS.RDY bit may be set to ‘1’ before namespace(s) are ready to be
accessed.
Setting this bit from a ‘0’ to a ‘1’ when the CSTS.RDY bit is a ‘1’ or clearing this
bit from a '1' to a '0' when the CSTS.RDY bit is cleared to '0' has undefined
results. The Admin Queue properties (AQA, ASQ, and ACQ) are only allowed
to be modified when this bit is cleared to ‘0’.
If an NVM Subsystem Shutdown is reported as in progress or is reported as
completed (i.e., the CSTS.ST bit is set to ‘1’, and the CSTS.SHST field is set to
01b or 10b), then:
• setting this bit from ‘0’ to ‘1’  modifies the field value but has no effect
(e.g., the controller does not respond by setting the CSTS.RDY bit to
‘1’); and
• clearing this bit from ‘1’ to ‘0’ resets the controller as defined by this
field.
Refer to section 3.6.3 for details on NVM Subsystem Shutdown functionality.
 Offset 1Ch: CSTS – Controller Status
Figure 42: Offset 1Ch: CSTS – Controller Status
Bits Type Reset1 Description
31:07 RO 0h Reserved
06 RO HwInit
Shutdown Type (ST): If CSTS.SHST is set to a non-zero value, then this bit indicates
the type of shutdown reported by CSTS.SHST.
If this bit is set to ‘1’, then CSTS.SHST is reporting the state of an NVM Subsystem
Shutdown and this bit remains set to ‘1’ until an NVM Subsystem Reset occurs.
If this bit is cleared to ‘0’, then CSTS.SHST is reporting the state of a controller
shutdown.
An NVM Subsystem Reset shall clear this bit to ‘0’. All other Controller Level Resets
shall not change the value of this bit.
If CSTS.SHST is cleared to 00b, then this bit should be ignored by the host.
