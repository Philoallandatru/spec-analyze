---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 215
headings: ["5.1.8.1 Command Completion"]
---

# Source page 215

NVM Express® Base Specification, Revision 2.1

193
Figure 181: Firmware Commit – Command Dword 10
Bits Description
02:00
Firmware Slot (FS): Specifies the firmware slot that shall be used f or the Commit Action, if applicable.
If the value specified is 0h, then the controller shall choose the firmware slot (i.e., slot 1 to slot 7) to use
for the operation.
 Command Completion
Upon completion of the Firmware Commit command, the controller posts a completion queue entry to the
Admin Completion Queue indicating the status for the command.
For Firmware Commit commands that specify activation of a new firmware image at the next Controller
Level Reset (i.e., the CA field was set to 001b or 010b) and complete with a status code value of 0h  (i.e.,
Success Completion) , a Controller Level Reset initiated by any of the methods defined in section 3.7.2
activates the specified firmware.
If the controller detects overlapping firmware/boot partition image update command sequences (refer to
section 1.5.41) of more than one firmware image and/or Boot Partition or the use of more than one controller
and/or Management Endpoint to update a single firmware image, then the results of that detection are
reported in Dword 0 of the completion queue entry as defined  in Figure 182. Refer to section 3.11 and
section 8.1.3.2.
If:
a) the Commit Action field is cleared to 000b (i.e., update the firmware image in the specified firmware
slot but do not activate);
b) the Firmware Slot field is set to the active firmware slot (refer to Figure 208); and
c) the controller supports greater than one firmware slot for a host to store firmware images,
then the controller may abort the command with a status code of Command Sequence Error. A host may
avoid this result by using a different firmware slot.
Figure 182: Firmware Commit – Completion Queue Entry Dword 0
Bits Description
31:02 Reserved
01:00
Multiple Update Detected (MUD): This field indicates if a controller detected overlapping firmware/boot
partition image update command sequences of Boot Partitions and/or firmware images (refer to section
3.11 and section 8.1.3.2). If the SMUD bit in the Firmware Update field of the Identify Controller data
structure is cleared to ‘0’, then this field shall be cleared to 00b.
This field is valid if the command is successful or aborted.
Bits Description
1
Management Endpoint FW Overlap (MEFWO): If this bit is set to ‘1’, then the controller
detected an overlapping firmware/boot partition image update command sequence due to
processing a command from a Management Endpoint. If this bit is cleared to ‘0’, then the
controller did not detect an overlapping firmware/boot partition image update command
sequence due to processing a command from a Management Endpoint.
0
Admin Submission Queue FW Overlap (ASQFWO): If this bit is set to ‘1’, then the
controller detected an overlapping firmware/boot partition image update command sequence
due to processing a command from an Admin Submission Queue on a controller. If this bit
is cleared to ‘0’, then the controller did not detect an overlapping firmware/boot partition
image update command sequence due to processing a command from an Admin
Submission Queue on a controller.

Firmware Commit command specific status values (i.e., SCT field set to 1h) are shown in Figure 183.
