---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 609
headings: []
---

# Source page 609

NVM Express® Base Specification, Revision 2.1

587
If a command supports selection of a UUID, then the UUID Selection Supported bit in the Commands
Supported and Effects data structure for that command (refer to Figure 210) shall be set to ‘1’. If a command
does not support selection of a UUID, then the UUID Selection Supported bit shall be cleared to ‘0’.
If the UUID Selection Supported bit is set to ‘1’ for one or more commands, then the UUID List bit in the
Controller Attributes field shall be set to ‘1’ (refer to Figure 312), and the controller shall support reporting
of a UUID List (refer to Figure 320).
If a command supports selection of a UUID, then that command contains a UUID Index field (refer to Figure
658).
Figure 658: UUID Index Field
Bits Description
6:0
UUID Index (UIDX): If this field is set to a non-zero value, then the value of this field is the index
of a UUID in the UUID List (refer to Figure 320) that is used by the command. If this field is cleared
to 0h, then no UUID index is specified.
If the UUID Index field specifies a valid UUID (i.e., the UUID Index field is set to a non -zero value and the
UUID at that index indicates a valid UUID) (refer to section 5.1.13.2.14), then the controller shall process
the command using the vendor specific information specified by that UUID. If the UUID Index field is cleared
to 0h, then the command does not specify a UUID.
If no UUID is specified by the command , then the controller shall process the command, returning vendor
specific information.
The controller shall abort the command with a status code of Invalid Field in Command if:
a) The controller does not support the UUID specified by the UUID Index for the specified information;
b) The UUID specified by the UUID Index is cleared to 0h; or
c) The UUID specified by the UUID Index is the NVMe Invalid UUID.
If a firmware image is activated that has a UUID List in which an entry is different from that of the previously-
active firmware image, then a host that is unaware of the change may issue a command with the UUID
index value for that entry. Such a command m ay produce unexpected results because the UUID specified
by that UUID Index has changed. To avoid this, vendors should follow the following revision guidelines for
UUID lists when constructing firmware images that support UUID selection:
a) Add UUIDs that are not supported in prior firmware image revisions to the end of the UUID List in
subsequent firmware image revisions;
b) Remove UUIDs that are supported in prior firmware image revisions by replacing the UUID with
the NVMe Invalid UUID in the same entry in the UUID list in subsequent firmware image revisions;
c) Do not replace the NVMe Invalid UUID with a valid UUID in the same UUID list entry in subsequent
firmware image revisions; and
d) Do not shorten or remove the UUID list in subsequent firmware image revisions.
In these guidelines, the terms “prior” and “subsequent” refer to a linear sequence of firmware versions (e.g.,
based on the date and time of the construction of the downloadable firmware image ).
Following these guidelines prevents the host from inadvertently specifying the wrong UUID because there
is at most one valid UUID for each entry in the UUID list. Hence a command that specifies a UUID Index
either specifies the intended UUID or is aborted because that entry in the UUID list is empty or contains the
NVMe Invalid UUID.
The controller shall require a reset to activate a downloaded firmware image (refer to section  5.1.8) if the
downloaded image reports a UUID list with at least one slot in which a valid UUID replaces the NVMe
Invalid UUID or a different valid UUID in the existing image. All controllers that are affected by the UUID list
change caused by activation of a d ownloaded firmware image shall be reset as part of activating that
downloaded firmware image.
The above requirements for a reset to activate a downloaded firmware image do not require the controller
to directly compare the UUID lists in the current and downloaded firmware images. For example, a vendor
