---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 153
headings: []
---

# Source page 153

NVM Express® Base Specification, Revision 2.1

131
2. The host submits a Firmware Commit command on that controller with a Commit Action of 011b
which specifies that the firmware image should be activated immediately without reset. The
downloaded firmware image shall replace the firmware image in the firmware slot. If no firmware
image was downloaded since the last reset or Firmware Commit command, (i.e., the first step was
skipped), then that controller shall verify and activate the  firmware image in the specified slot. If
that controller starts to activate t he firmware image, any controllers affected by the new firmware
image send a Firmware Activation Starting asynchronous event to the host if Firmware Activation
Notices are enabled (refer to Figure 391):
a. The Firmware Commit command may also be used to activate a firmware image associated
with a previously committed firmware slot;
3. The controller completes the Firmware Commit command. The following actions are taken in
certain error scenarios:
a. If the firmware image is invalid, then the controller aborts the command with an appropriate
status code (e.g., Invalid Firmware Image);
b. If the firmware activation was not successful because a Controller Level Reset is required to
activate this firmware  image, then the controller aborts the command with a status code of
Firmware Activation Requires Controller Level Reset  and the firmware image is applied at the
next Controller Level Reset;
c. If the firmware activation was not successful because an NVM Subsystem Reset is required to
activate this firmware  image, then the controller aborts the command with a status code of
Firmware Activation Requires NVM Subsystem Reset and the firmware image is applied at the
next NVM Subsystem Reset;
d. If the firmware activation was not successful because a Conventional Reset is required to
activate this firmware  image, then the controller aborts the command with a status code of
Firmware Activation Requires Conventional Reset and the firmware image is applied at the
next Conventional Reset; and
e. If the firmware activation was not successful because the firmware activation time requires
more time than the time reported by  the MTFA field in the Identify Controller data structure,
then the controller aborts the command with a status code of  Firmware Activation Requires
Maximum Time Violation. In this case, the firmware image was committed to the specified
firmware slot. To activate that firmware image, the host may issue a Firmware Commit
command that specifies:
i. a Commit Action set to 010b (i.e., activate using a Controller Level Reset); and
ii. the same firmware slot.
If the controller transitions to the D3 cold state (refer to the PCI Express Base Specification) after the
submission of a Firmware Commit command that attempts to activate a firmware image and before the
completion of that command, then the controller may resume operation with either the firmware i mage
active at the time the Firmware Commit command was submitted or the firmware image that was activated
by that command.
If the firmware image is not able to be successfully loaded, then the controller shall revert to the firmware
image present in the most recently activated firmware slot or the baseline read -only firmware image, if
available, and indicate the failure as an asynchronous event wit h a Firmware Image Load Error.  If the
controller changes (e.g., reverts) the firmware image while a sanitize operation is in progress, then that
sanitize operation fails (refer to section 8.1.24.3).
If a host overwrites (i.e., updates) the firmware image in the active firmware slot, then the previously active
firmware image may no longer be available. As a result, any action (e.g., power cycling the controller) that
requires the use of that firmware slot may instead use the firmware image that is currentl y in that firmware
slot. If the firmware image that is currently in that firmware slot would be activated by such an action
replacing the currently active firmware, then:
• this is a pending firmware activation with reset; and
• a controller aborts subsequent Sanitize commands with a status code indicating the appropriate
reset required to activate the pending activation as defined in section 5.1.22.
