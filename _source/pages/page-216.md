---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 216
headings: ["5.1.9 Firmware Image Download command"]
---

# Source page 216

NVM Express® Base Specification, Revision 2.1

194
Figure 183: Firmware Commit – Command Specific Status Values
Value Description
06h Invalid Firmware Slot:  The firmware slot indicated is invalid  or read only . This error is indicated if the
firmware slot exceeds the number supported.
07h
Invalid Firmware Image: The firmware image specified for activation is:
• invalid and not loaded by the controller; or
• the specified firmware slot does not contain a firmware image.
0Bh
Firmware Activation Requires Conventional Reset: The firmware commit was successful; however,
activation of the firmware image requires a Conventional Reset. (refer to the NVM Express NVMe over
PCIe Transport Specification). If a Function Level Reset (refer to the NVM Express NVMe over PCIe
Transport Specification) or Controller Reset occurs prior to a Conventional Reset , the controller shall
continue operation with the currently executing firmware image.
10h
Firmware Activation Requires NVM Subsystem Reset: The firmware commit was successful;
however, activation of the firmware image requires an NVM Subsystem Reset. If any other type of
Controller Level Reset occurs prior to an NVM Subsystem Reset, the controller shall continue operation
with the currently executing firmware image.
11h
Firmware Activation Requires Controller Level Reset: The firmware commit was successful; however,
the firmware image specified does not support being activated without a Controller Level R eset. The
firmware image shall be activated at the next Controller Level Reset. This status code should be returned
only if the Commit Action field in the Firmware Commit command is set to 011b (i.e., activate immediately).
12h
Firmware Activation Requires Maximum Time Violation: The firmware commit was successful;
however, the firmware image was not activated. Activation of the specified firmware image requires more
time than the amount indicated in the Maximum Time for Firmware Activation (MTFA) field in the Identify
Controller data structure (refer to Figure 312). To activate the firmware image committed to the specified
firmware slot, a Firmware Commit command specifying a Commit Action set to 010b may be issued as
described in section 3.11.
13h Firmware Activation Prohibited:  The image specified is being prohibited from activation by the
controller for vendor specific reasons (e.g., controller does not support down revision firmware).
14h Overlapping Range : This error is indicated if the controller detects that the firmware image has
overlapping ranges.
1Eh Boot Partition Write Prohibited: This error is indicated if a command attempts to modify a Boot Partition
while write locked (refer to section 8.1.3.3).
 Firmware Image Download command
The Firmware Image Download command is used to download all or a portion of an image for a future
update to the controller.  The Firmware Image Download command may be submitted while other
commands on the Admin Submission Queue or I/O Submission Queues are outstanding.  The Firmware
Image Download command downloads a new image (in whole or in part) to the controller.
The image may be constructed of multiple pieces that are individually downloaded with separate Firmware
Image Download commands.  Each Firmware Image Download command includes a Dword Offset and
Number of Dwords that specify a dword range. The host software should ensure that image pieces do not
have dword ranges that overlap and that the NUMD field and OFST field meet the alignment and granularity
requirements indicated in the FWUG field (refer to Figure 312). Firmware portions may be submitted out of
order to the controller. Host software shall submit image portions in order when updating a Boot Partition.
If ranges overlap, the controller may return an error of Overlapping Range.
The new firmware image is not activated as part of the Firmware Image Download command. Refer to
section 3.11 for details on the firmware update process. The firmware update process does not modify the
contents of Boot Partitions. Refer to section 8.1.3.2 for details on the Boot Partition update process.
Host software should not overlap command sequences that update Boot Partitions and/or firmware images
(refer to section 3.11 and section 8.1.3.2).
After downloading an image, host software issues a  Firmware Commit command before downloading
another image. Processing of the first Firmware Image Download command after completion of a Firmware
Commit command shall cause the controller to discard all remaining portion(s), if any, of downloaded
