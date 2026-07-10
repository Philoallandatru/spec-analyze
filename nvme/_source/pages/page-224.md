---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 224
headings: []
---

# Source page 224

NVM Express® Base Specification, Revision 2.1

202
Figure 196: Get Log Page – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 197: Get Log Page – Command Dword 10
Bits Description
31:16
Number of Dwords Lower (NUMDL): This field specifies the least significant 16 bits of the number of
dwords to return unless otherwise specified . If host software specifies a size larger than the log page
requested, the controller returns the complete log page with undefined results for dwords beyond the end
of the log page. The combined NUMDL and NUMDU fields form a 0’s based value.
15
Retain Asynchronous Event (RAE):  This bit specifies whether to retain or clear an Asynchronous
Event. If this bit is cleared to ‘0’, then the corresponding Asynchronous Event is cleared by the controller
upon successful command completion. If this bit is set to ‘1’, then the corresponding Asynchronous Event
is retained (i.e., not cleared) by the controller upon command completion.
If the command does not complete successfully, the Asynchronous Event shall be retained by the
controller.
Host software should clear this bit to ‘0’ for log pages that are not used with Asynchronous Events. Refer
to section 5.1.2.
Refer to the NVM Express Admin Command Set section of the NVM Express Management Interface
Specification for the behavior associated with this bit when the Get Log Page command is received on a
Management Endpoint.
14:08 Log Specific Parameter (LSP): If not defined for the log specified by the Log Page Identifier field, this
field is reserved.
07:00 Log Page Ide ntifier (LID): This field specifies the identifier of the log page to retrieve  (refer to Figure
202).

Figure 198: Get Log Page – Command Dword 11
Bits Description
31:16 Log Specific Identifier (LSI): This field specifies an identifier that is required for a particular log page.
If not defined for the log page specified by the Log Page Identifier field, this field is reserved.
15:00 Number of Dwords (NUMDU): This field specifies the most significant 16 bits of the number of dwords
to return unless otherwise specified.
