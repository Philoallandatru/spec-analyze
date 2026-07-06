---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 152
headings: ["3.10 Privileged Actions", "3.11 Firmware Update Process"]
---

# Source page 152

NVM Express® Base Specification, Revision 2.1

130
section 9.6 to avoid possible data corruption caused by interaction between outstanding commands and
subsequent commands submitted by that host to another controller.
3.10 Privileged Actions
Privileged actions are actions (e.g., command, property write) that affect or have the potential to affect the
state beyond the controller and attached namespaces.
Examples of privileged actions are:
• Admin commands includ ing Namespace Management, Namespace Attachment, Virtualization
Management, Format NVM, Set Features with Feature Identifier 17h (i.e., Sanitize Config, refer to
section 5.1.25.1.15), Sanitize, Capacity Management, Controller Data Queue, Migration Receive,
Migration Send, Track Send, and Track Receive;
• Property Writes including NVM Subsystem Reset; and
• Some Vendor specific commands and properties.
3.11 Firmware Update Process
The process for a firmware update to be activated in a domain (refer to section 3.2.5) by a reset is:
1. The host issues a Firmware Image Download command to download the firmware image to a
controller. There may be multiple portions of the firmware image to download, thus the offset for
each portion of the firmware image being downloaded on that controller is specified in the Firmware
Image Download command. The data provided in the Firmware Image Download command should
conform to the Firmware Update Granularity indicated in the Identify Controller data structure or
the firmware update may fail;
2. After the firmware is downloaded to that controller, the next step is for the host to submit a Firmware
Commit command to that controller. The Firmware Commit command verifies that the last firmware
image downloaded is valid and commits that firmware image to the firmware slot indicated for future
use. A firmware image that does not start at offset zero, contains gaps, or contains overlapping
regions is considered invalid. A controller may employ additional vendor specific means (e.g.,
checksum, CRC, cryptographic hash, or a digital signature) to determine the validity of a firmware
image:
a. The Firmware Commit command may also be used to activate a firmware image associated
with a previously committed firmware slot;
3. The host performs an action that results in a Controller Level Reset (refer to section 3.7.2) on that
controller to cause the firmware image specified in the Firmware Slot field in the Firmware Commit
command to be activated:
a. In some cases, a Conventional Reset (refer to the NVM Express NVMe over PCIe Transport
Specification) or NVM Subsystem Reset is required to activate a firmware image. This
requirement is indicated by Firmware Commit command specific status (refer to section
5.1.8.1);
and
4. After the reset has completed, host software re-initializes the controller. This includes re-allocating
I/O Submission and Completion Queues. Refer to sections 3.5.1 and 3.5.2.
The process for a firmware update to be activated on a domain without a reset is:
1. The host issues a Firmware Image Download command to download the firmware image to a
controller. There may be multiple portions of the firmware image to download, thus the offset for
each portion of the firmware image being downloaded on that controller is specified in the Firmware
Image Download command. The data provided in the Firmware Image Download command should
conform to the Firmware Update Granularity indicated in the Identify Controller data structure or
the firmware update may fail;
