---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 469
headings: ["5.3.9.2 Command Completion", "5.3.10 Send Discovery Log Page command"]
---

# Source page 469

NVM Express® Base Specification, Revision 2.1

447
Upon successful completion of a Manage Exported Port command with a Delete operation the specified
Exported Port ID has been removed from the specified Exported NVM Subsystem.
 Command Completion
Upon completion of the Manage Exported Port command, the controller posts a completion queue entry to
the Admin Completion queue indicating the status of the command. Refer to section  8.3.3 for usages.
 Send Discovery Log Page command
The Send Discovery Log Page (SDLP) command is used by a CDC to send a discovery log page to a pull
model DDC. This command uses the Data Pointer, Command Dword 10, Command Dword 11, Command
Dword 12, and Command Dword 13, as shown in Figure 533, Figure 534, Figure 535, Figure 536, and
Figure 537 respectively.
Figure 533: Send Discovery Log Page (SDLP) – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 534: Send Discovery Log Page (SDLP) – Command Dword 10
Bits Description
31:30
Requested Log Page Status (RLPS):  This field specified the status of the requested log page. Defined
values are:
Value Definition
00b Valid Log Page: the requested log page is carried in the command.
01b Invalid Log Page: The requested log page is invalid or not supported.
10b Not Allowed Log Page: The requested log page is not allowed to be transferred by an SDLP
command (refer to Figure 538).
11b Not Successful: Retrieving the requested log page failed. Further details are provided in the
SCT field and the SC field.

29:28 Reserved
27:25 Status Code Type (SCT): Refer to Figure 100.
24:17 Status Code (SC): Refer to Figure 100.
16:15 Reserved
14:08 Transferred Log Specific Parameter (TLSP):  If not defined for the log page specified by the Log Page
Identifier field, this field is reserved.
07:00 Transferred Log Page Identifier (TLID): This field specifies the identifier of the sent log page.

Figure 535: Send Discovery Log Page (SDLP) – Command Dword 11
Bits Description
31:00 Number of Dwords (NDWS): This field specifies the number of transferred dwords.

Figure 536: Send Discovery Log Page (SDLP) – Command Dword 12
Bits Description
31:00 Transferred Log Page Offset Lower (TLPOL): This field specifies the least-significant 32 bits of the offset
of the transferred log page.

Figure 537: Send Discovery Log Page (SDLP) – Command Dword 13
Bits Description
31:00 Transferred Log Page Offset Upper (TLPOU): This field specifies the most-significant 32 bits of the offset
of the transferred log page.
