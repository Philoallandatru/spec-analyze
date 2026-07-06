---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 470
headings: []
---

# Source page 470

NVM Express® Base Specification, Revision 2.1

448
The transferred log page has the same format of the correspondent discovery log page retrieved by a Get
Log Page command (refer to section 5.1.12). The discovery log pages allowed to be transferred by a SDLP
command are shown in Figure 538.
Figure 538: Send Discovery Log Page (SDLP) – Allowed Log Page Identifiers
Log Page Identifier Log Page Name
70h Discovery
71h Host Discovery
72h AVE Discovery
If a not allowed log page is requested, the CDC shall set the RLPS field (refer to Figure 534) to 02h (i.e.,
Not Allowed Log Page) and transfer no log page.
The pull model DDC receiving a SDLP command may request the CDC to resend the requested log page
when that log page is updated on the CDC. This is done through the LPUR bit of the SDLP Completion
Queue Entry Dword 0, as shown in Figure 539.
Figure 539: Send Discovery Log Page (SDLP) – Completion Queue Entry Dword 0
Bits Description
31
Log Page Update Registration (LPUR): This bit specifies if the CDC shall resend the requested log
page when that log page is updated on the CDC. If this bit is set to ‘1’, then the CDC shall issue to the
requesting pull model DDC a Send Discovery Log Page command including the requested log page
when that log page is updated on the CDC . If this bit is cleared to ‘0’, then the CDC is not required to
issue a Send Discovery Log Page command when that log page is updated on the CDC. This
registration is retaine d until the CDC issues the subsequent Send Discovery Log Page command to
that requesting pull model DDC.
30:00 Reserved
