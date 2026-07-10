---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 235
headings: ["5.1.12.1.4 Firmware Slot Information (Log Page Identifier 03h)", "5.1.12.1.5 Changed Attached Namespace List (Log Page Identifier 04h)"]
---

# Source page 235

NVM Express® Base Specification, Revision 2.1

213
5.1.12.1.4 Firmware Slot Information (Log Page Identifier 03h)
This log page is used to describe the firmware revision stored in each firmware slot supported. The firmware
revision is indicated as an ASCII string.  The log page also indicates the active slot number.  The log page
returned is defined in Figure 208.
Figure 208: Firmware Slot Information Log Page
Bytes Description
00
Active Firmware Info (AFI): Specifies information about the active firmware revision.
Bits Description
7 Reserved
6:4
Next Active Firmware Slot (NAFS): This field indicates the firmware slot that is going to be
activated at the next Controller Level Reset. If this field is cleared to 0h, then the controller
does not indicate the firmware slot that is going to be activated at the next Controller Level
Reset.
3 Reserved
2:0 Current Active Firmware Slot (CAFS): This field indicates the firmware slot from which the
actively running firmware revision was loaded

07:01 Reserved
15:08
Firmware Revision for Slot 1 (FRS1): Contains the revision of the firmware downloaded to firmware
slot 1. If no valid firmware revision is present or if this slot is unsupported, this field shall be cleared to
0h.
23:16
Firmware Revision for Slot 2 (FRS2): Contains the revision of the firmware downloaded to firmware
slot 2. If no valid firmware revision is present or if this slot is unsupported, this field shall be cleared to
0h.
31:24
Firmware Revision for Slot 3 (FRS3): Contains the revision of the firmware downloaded to firmware
slot 3. If no valid firmware revision is present or if this slot is unsupported, this field shall be cleared to
0h.
39:32
Firmware Revision for Slot 4 (FRS4): Contains the revision of the firmware downloaded to firmware
slot 4. If no valid firmware revision is present or if this slot is unsupported, this field shall be cleared to
0h.
47:40
Firmware Revision for Slot 5 (FRS5): Contains the revision of the firmware downloaded to firmware
slot 5. If no valid firmware revision is present or if this slot is unsupported, this field shall be cleared to
0h.
55:48
Firmware Revision for Slot 6 (FRS6): Contains the revision of the firmware downloaded to firmware
slot 6. If no valid firmware revision is present or if this slot is unsupported, this field shall be cleared to
0h.
63:56
Firmware Revision for Slot 7 (FRS7): Contains the revision of the firmware downloaded to firmware
slot 7. If no valid firmware revision is present or if this slot is unsupported, this field shall be cleared to
0h.
511:64 Reserved
5.1.12.1.5 Changed Attached Namespace List (Log Page Identifier 04h)
This log page is used to describe  changes to attached namespaces for this controller, since the last time
that this log page was read, that:
a) have changed information in their Identify Namespace data structures (refer to section 1.5.49);
b) were previously unattached to the controller and have since been attached to the controller ;
c) were previously attached to the controller and have since been detached from the controller; and
d) were deleted.
The log page contains a Namespace List with up to 1 ,024 entries. If more than 1 ,024 namespaces have
changed attributes since the last time the log page was read, the first entry in the log page shall be set to
FFFFFFFFh and the remainder of the list shall be zero filled.
