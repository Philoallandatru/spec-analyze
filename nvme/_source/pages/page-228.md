---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 228
headings: []
---

# Source page 228

NVM Express® Base Specification, Revision 2.1

206
Figure 203: Supported Log Pages Log Page
Bytes Description
3:0 Log Page Identifier Supported 0  (LIDS0): Contains the LID Supported and Effects data structure
(refer to Figure 204) for the LID 0h.
7:4 Log Page Identifier Supported 1  (LIDS1): Contains the LID Supported and Effects data structure
(refer to Figure 204) for the LID 1h.
… …
1019:1016 Log Page Identifier Supported 254  (LIDS254): Contains the LID Supported and Effects data
structure (refer to Figure 204) for the LID FEh.
1023:1020 Log Page Identifier Supported 255  (LIDS255): Contains the LID Supported and Effects data
structure (refer to Figure 204) for the LID FFh.

Figure 204: LID Supported and Effects Data Structure
Bits Description
31:16 LID Specific Parameter (LIDSP): This field is specific to the log page identifier a nd if not defined for
the log specified by the Log Page Identifier field, this field is reserved.
15:2 Reserved
1
Index Offset Supported (IOS): If this bit is set to ‘1’, then the controller supports an index offset for
this LID in a Get Log Page command (i.e., the OT bit in the Get Log Page command is allowed to be
set to ‘1’). If this bit is cleared to ‘0’, then the controller does not support an index offset for this LID in
a Get Log Page command (i.e., the OT bit in the Get Log Page command is only allowed to be cleared
to ‘0’).
If the controller does not support extended data for the Get Log Page command (refer to the Log Page
Extended Data Support (SPEDS) bit in the LPA field in Figure 312), then this bit shall be cleared to ‘0’.
0
LID Supported (LSUPP): If this bit is set to ‘1’, then the controller supports this LID for a Get Log Page
command. If this bit is cleared to ‘0’, then the controller does not support this LID for a Get Log Page
command, and the host should ignore other fields in this data structure.
Refer to section 3.1.3.5 for the LID support requirements for each controller type.
