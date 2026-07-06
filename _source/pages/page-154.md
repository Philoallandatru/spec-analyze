---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 154
headings: []
---

# Source page 154

NVM Express® Base Specification, Revision 2.1

132
Host software should not overlap firmware/boot partition image update command sequences (refer to
section 1.5.41). During a firmware image update command sequence, if a Firmware Image Download
command or a Firmware Commit command is submitted for another firmware/boot partition image update
command sequence, the results of both that command and the in -progress firmw are image update are
undefined.
Host software should use the same controller or Management Endpoint (refer to the NVM Express
Management Interface Specification) for all commands that are part of a firmware image update command
sequence. If the commands for a single firmware/boot partition image update command sequence are
submitted to more than one controller and/or Management Endpoint, the controller may abort the Firmware
Commit command with Invalid Firmware Image status.
After downloading a  firmware image, host software issues a Firmware Commit command before
downloading additional firmware images. Processing of the first Firmware Image Download command after
completion of a Firmware Commit command shall cause the controller to discard remaining portions, if any,
of downloaded images. If a reset occurs between a firmware download and completion of the Firmware
Commit command, then the controller shall discard all portion(s), if any, of downloaded images.
