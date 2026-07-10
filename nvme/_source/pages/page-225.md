---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 225
headings: []
---

# Source page 225

NVM Express® Base Specification, Revision 2.1

203
Figure 199: Get Log Page – Command Dword 12
Bits Description
31:00
Log Page Offset Lower (LPOL): The log page offset specifies the location within a log page to start
returning data from unless otherwise specified.
If the OT bit is cleared to ‘0’, then:
a) This field specifies the least significant 32 bits of the log page offset. The offset shall be dword
aligned, indicated by bits 1:0 being cleared to 00b;
b) The controller is not required to check that bits 1:0 are cleared to 00b. The controller may report
an error of Invalid Field in Command if bits 1:0 are not cleared to 00b. If the controller does not
report an error of Invalid Field in Command, then the controller shall operate as if bits 1:0 are
cleared to 00b; and
c) If the host specifies an offset (i.e., LPOL and LPOU) that is greater than the size of the log page
requested (e.g., a log page containing 100 bytes is requested starting at offset 200), then the
controller shall abort the command with a status of Invalid Field in Command.
If the OT bit is set to ‘1’, then:
a) This field specifies the least significant 32 bits of the index into the list of data structures in the
log page;
b) If the host specifies an index (i.e., LPOL and LPOU) that is greater than the number of entries
in the list of data structures in the log page requested (e.g., a log page containing 100 data
structures is requested starting at index 200), then the controller shall abort the command with
a status code of Invalid Field in Command;
c) If the IOS bit is cleared to ‘0’ in the LID Supported and Effects data structure for the specified
LID in the Supported Log Pages log page, then the controller shall abort the command with a
status code of Invalid Field in Command; and
d) Each log page that supports the use of an index offset value defines the contents of an entry
for the purposes of indexing into that log page.

Figure 200: Get Log Page – Command Dword 13
Bits Description
31:00
Log Page Offset Upper (LPOU): This field specifies the most significant 32 bits of either the log page
offset or the index into the list of data structures unless otherwise specified. Refer to the Log Page Offset
Lower definition.
If the controller supports selection of a UUID by the Get Log Page command (refer to Figure 202 and section
8.1.28), then Command Dword 14 is used to specify a UUID Index value (refer to Figure 201).
Figure 201: Get Log Page – Command Dword 14
Bits Description
31:24
Command Set Identifier  (CSI): This field specifies the I/O Command Set (r efer to Figure 311) to be
used by the command. This field is log page specific; refer to Figure 202 for log pages that use this field.
This field shall be ignored by the controller if the specified log page does not support the use of this field
(refer to Figure 202) or if the CC.CSS field is not set to 110b.
23
Offset Type (OT): If this bit is set to ‘1’, then the Log Page Offset Lower field and the Log Page Offset
Upper field specify the index into the list of data structures in the log page to be returned. If  this bit is
cleared to ‘0’, then the Log Page Offset Lower field and the Log Page Offset Upper field specify the byte
offset into the log page to be returned.
22:07 Reserved
06:00 UUID Index (UIDX): Refer to Figure 658.
Figure 202 defines the log pages that are able to be retrieved with the Get Log Page command and the
scope of the information that is returned in those log pages. Section 5.1.12.1 describes log pages that are
common to all transport models . Section 5.1.12.2 describes log pages that are specific to the Memory -
based transport model. Section 5.1.12.3 describes log pages  that are specific to the Message -based
transport model.
