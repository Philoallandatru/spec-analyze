---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 221
headings: []
---

# Source page 221

NVM Express® Base Specification, Revision 2.1

199
Figure 191: Get Features – Data Pointer
Bits Description
127:00
Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field. If no data structure is used as part of the specified feature (refer to the FID field), then this
field shall be ignored by the controller.

Figure 192: Get Features – Command Dword 10
Bits Description
31:11 Reserved
10:08
Select (SEL): This field specifies the attribute of the value requested in the returned data:
Select Description
000b Current
001b Default
010b Saved
011b Supported Capabilities
100b to 111b Reserved
Refer to section 5.1.11.1 and section 4.4 for details on the data returned in each case.
The controller indicates in  the Save and Select Feature Support (SSFS)  bit of the Optional NVM
Command Support field of the Identify Controller data structure in Figure 312 whether this field is
supported.
If a Get Features command is received with the Select field set to 010b (i.e., saved) and the controller
does not support the Feature Identifier being saved or does not currently have any saved values, then
the controller shall operate as if the Select field is set to 001b (i.e., default).
07:00 Feature Identifier (FID): This field specifies the identifier of the Feature for which to provide data.
If the controller supports selection of a UUID by the Get Features command (refer to Figure 385 and section
8.1.28) and the controller supports selection of a UUID for the specified vendor specific Feature Identifier
(refer to Figure 385), then Command Dword 14 is used to specify a UUID Index value (refer to Figure 193).
If the controller does not support selection of a UUID by the Get Features command or the controller does
not support selection of a UUID for the specified vendor specific Feature Identifier, then Command Dword
14 does not specify a UUID Index value.
Figure 193: Get Features – Command Dword 14
Bits Description
31:07 Reserved
06:00 UUID Index (UIDX): Refer to Figure 658.
Figure 194 describes the Feature Identifiers whose attributes may be retrieved using the Get Features
command. The definition of the attributes returned and the associated format is specified in the section
indicated.
Figure 194: Get Features – Feature Identifiers
Description Feature
Identifier
Section Defining
Format of Attributes
Returned
Arbitration  01h 5.1.25.1.1
Power Management 02h 5.1.25.1.2
Temperature Threshold 04h 5.1.25.1.3
Volatile Write Cache 06h 5.1.25.1.4
Number of Queues 07h 5.1.25.2.1
