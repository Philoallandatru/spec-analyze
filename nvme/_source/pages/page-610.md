---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 610
headings: ["8.1.28.3 UUIDs for Vendor Specific Information Examples", "8.1.28.3.1 Vendor Specific Log Page Example", "8.1.28.3.2 Vendor Specific Feature Example"]
---

# Source page 610

NVM Express® Base Specification, Revision 2.1

588
could use a vendor-specific major.minor firmware image revision numbering system (e.g., 3.5, 4.1) where
all downloadable firmware images with the same major revision number follow the above guidelines. In that
scenario, the controller is able to meet these  reset requirements by requiring a reset if the downloaded
firmware image and the currently executing firmware have different major revision numbers.
 UUIDs for Vendor Specific Information Examples
This section includes examples of the use of UUIDs to select vendor specific information.
8.1.28.3.1 Vendor Specific Log Page Example
If entity C and entity V create different definitions for a vendor specific log page having the same log page
identifier (e.g., D0h), then each assigns a UUID to distinguish their definition (e.g., entity V assigns UUID V
and entity C assigns UUID C).
A controller supporting both definitions of the log page:
a) Sets the UUID List bit to ‘1’ in the CTRATT field of the Identify Controller data structure (refer to
Figure 312);
b) Sets the UUID Selection Supported bit to ‘1’ in the Commands Supported and Effects data structure
(refer to Figure 210) corresponding to the Get Log Page command; and
c) Reports both UUID V and UUID C in the UUID list (refer to Figure 320).
A host requesting the log page defined by entity C:
1) Determines the index of UUID C in the UUID list;
2) Sets the Log Page Identifier field of the Get Log Page command to D0h; and
3) Sets the UUID Index field of the Get Log Page command to the index of UUID C.
A host requesting the log page defined by entity V:
1) Determines the index of UUID V in the UUID list;
2) Sets the Log Page Identifier field of the Get Log Page command to D0h; and
3) Sets the UUID Index field of the Get Log Page command to the index of UUID V.
A host not specifying the definition of the log page clears the UUID Index field to 0h. The selection of the
log page definition returned by the controller is vendor specific (e.g., the controller may select any definition
for the returned data).
8.1.28.3.2 Vendor Specific Feature Example
If entity C and entity V create different definitions for a vendor specific feature having the same Feature
Identifier (e.g., F1h), then each assigns a UUID to distinguish their definitions (e.g., entity V assigns UUID
V and entity C assigns UUID C).
A controller supporting both definitions of the feature for the Get Features command:
a) Sets the UUID List bit to ‘1’ in the CTRATT field of the Identify Controller data structure (refer to
Figure 312);
b) Sets the UUID Selection Supported bit to ‘1’ in the Commands Supported and Effects data structure
(refer to Figure 210) corresponding to the Get Features command;
c) Sets the UUID Selection Supported bit to ‘1’ in the FID Supported and Effects log page (refer to
Figure 262); and
d) Reports both UUID V and UUID C in the UUID list (refer to Figure 320).
A host retrieving the attributes of the feature defined by entity C:
1) Determines the index of UUID C in the UUID list;
2) Sets the Feature Identifier field of the Get Features command to F1h; and
3) Sets the UUID Index field of the Get Features command to the index of UUID C.
A host retrieving the attributes of the feature defined by entity V:
1) Determines the index of UUID V in the UUID list;
