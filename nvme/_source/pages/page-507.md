---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 507
headings: ["8.1.3.3 Boot Partition Write Protection"]
---

# Source page 507

NVM Express® Base Specification, Revision 2.1

485
1. The host issues a Firmware Image Download command to download the contents of the Boot
Partition to a controller. There may be multiple portions of the Boot Partition to download, thus the
offset for each portion of the Boot Partition being downloaded is specified in the Firmware Image
Download command. A host shall send the Boot Partition image in order starting with the beginning
of the Boot Partition;
2. The host transitions the Boot Partition that is to be updated to the Write Unlocked State  (refer to
section 8.1.3.3);
3. The host submits a Firmware Commit command  (refer to section  5.1.8) on that controller  with a
Commit Action of 110b which specifies that the downloaded image replaces the contents of the
Boot Partition specified in the Boot Partition ID field;
4. The controller completes the Firmware Commit command. The following actions are taken in
certain error scenarios:
a. If the firmware activation was not successful because the Boot Partition could not be written,
then the controller reports an error of Boot Partition Write Prohibited;
5. (Optional) The host reads the contents of the Boot Partition to verify they are correct (refer to section
8.1.3.1). A host updates the active Boot Partition ID by issuing a Firmware Commit command with
a Commit Action of 111b; and
6. The host transitions the Boot Partition to either the Write Locked State or Write Locked Until Power
Cycle State to prevent further modification (refer to section 8.1.3.3).
If an internal error, reset, or power loss condition occurs while committing the downloaded image to a Boot
Partition, the contents of the Boot Partition may contain the old contents, new contents, or a mixture of both.
Host software should verify the cont ents of a Boot Partition before marking that Boot Partition active to
ensure the active Boot Partition is stable.
Host software should not read the contents of a Boot Partition while writing to the Boot Partition. The
controller may return a combination of new and old data if the host attempts to perform a Boot Partition
read operation while overwriting the contents.
Host software should not overlap firmware/boot partition image update command sequences (refer to
section 1.5.41). During a boot partition image update command sequence, if a Firmware Image Download
command or a Firmware Commit command is submitted for another firmware/boot partition image update
command sequence, the results of both that command and the in -progress firmware image update are
undefined.
Host software should use the same controller or Management Endpoint (refer to the NVM Express
Management Interface Specification ) for all commands that are part of a boot partition image update
command sequence. If the commands for a single firmware/boot partition image update command
sequence are submitted to more than one controller and/or Management Endpoint, the controller may abort
the Firmware Commit command with Invalid Firmware Image status.
 Boot Partition Write Protection
A controller that supports both Boot Partitions and RPMB shall support at least one of the following Boot
Partition write protection mechanisms:
• Set Features Boot Partition Write Protection (refer to section 8.1.3.3.1); or
• RPMB Boot Partition Write Protection (refer to section 8.1.3.3.2).
It is not recommended that a controller support both Set Features Boot Partition Write Protection and RPMB
Boot Partition Write Protection.
A controller that supports Boot Partitions and does not support RPMB shall support Set Features Boot
Partition Write Protection.
A host is able to determine whether Set Features Boot Partition Write Protection is supported by checking
the value of the Set Features Boot Partition Write Protection Support  bit of the Boot Partition Capabilities
field in the Identify Controller Data Structure (refer to Figure 312).
A host is able to determine whether RPMB Boot Partition Write Protection is supported by checking the
RPMB Boot Partition Write Protection Support field  of the Boot Partition Capabilities field in the Identify
