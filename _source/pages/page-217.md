---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 217
headings: ["5.1.9.1 Command Completion", "5.1.10 Format NVM command"]
---

# Source page 217

NVM Express® Base Specification, Revision 2.1

195
images. If a Controller Level Reset occurs between a firmware download and completion of the Firmware
Commit command, then the controller shall discard all portion(s), if any, of downloaded images.
The Firmware Image Download command uses the Data Pointer, Command Dword 10, and Command
Dword 11 fields. All other command specific fields are reserved.
Figure 184: Firmware Image Download – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the location where data should be transferred from.  Refer to
Figure 92 for the definition of this field.

Figure 185: Firmware Image Download – Command Dword 10
Bits Description
31:00
Number of Dwords (NUMD): This field specifies the number of dwords to transfer for this portion of the
firmware. This is a 0’s based value. If the value specified in this field does not meet the requirement
indicated by the FWUG field (refer to  Figure 312), then the controller may abort the command   with a
status code of Invalid Field in Command.

Figure 186: Firmware Image Download – Command Dword 11
Bits Description
31:00
Offset (OFST): This field specifies the number of dwords offset from the start of the firmware image
being downloaded to the controller.  The offset is used to construct the complete firmware image when
the firmware is downloaded in multiple pieces. The piece corresponding to the start of the firmware image
shall have an Offset of 0h. If the value specified in this field does not meet the requirement indicated by
the FWUG field (refer to Figure 312), then the controller may abort the command  with a status code of
Invalid Field in Command.
 Command Completion
Upon completion of the Firmware Image Download command, the controller posts a completion queue
entry to the Admin Completion Queue.  Firmware Image Download command specific status values are
defined in Figure 187.
Figure 187: Firmware Image Download – Command Specific Status Values
Value Description
14h
Overlapping Range:  This error is indicated if the  controller detects that the  firmware image has
overlapping ranges. This error may indicate that the granularity or alignment of the firmware image
downloaded does not conform to the Firmware Update Granularity field indicated in the Identify
Controller data structure.
If the controller detects overlapping firmware/boot partition image update command sequences (refer to
section 1.5.41) of more than one firmware image and/or Boot Partition or the use of more than one controller
and/or Management Endpoint to update a single firmware image, then the results of that detection are
reported in Dword 0 of the completion queue entry as defined  in Figure 182. Refer to section 3.11 and
section 8.1.3.2.
 Format NVM command
The Format NVM command is used to low level format the NVM media. This command is used by the host
to change the attributes of the NVM media (e.g., the LBA data size and/or metadata size  for the NVM
Command Set). A low level format may destroy all data and metadata associated with all namespaces that
contain formatted storage or only the specific namespace associated with the command (refer to the Format
NVM Attributes field in the Identify Controller data structure , Figure 312). After the Format NVM command
successfully completes, the controller shall not return any user data that was previously contained in an
affected namespace.
