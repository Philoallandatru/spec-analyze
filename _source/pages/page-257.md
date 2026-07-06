---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 257
headings: []
---

# Source page 257

NVM Express® Base Specification, Revision 2.1

235
Figure 226: Persistent Event Log Page
Bytes Description
377:374
Reporting Context Information (RCI): This field contains information about the persistent event
log reporting context.
Bits Description
31:19 Reserved
18
Reporting Context Exists (RCE): This bit indicates the persistent event log reporting
context. If this bit is set to ‘1’, then a persistent event log reporting context already
existed when the Get Log Page command that requested this log page was
processed. If this bit is cleared to ‘0’, then a persistent event log reporting context did
not already exist when the Get Log Page command that requested this log page was
processed.
17:16
Reporting Context Port Identifier Type (RCPIT):  If the RCE bit is set to ‘1’, then
this field indicates the type of port identifier reported in the Reporting Context Port
Identifier (RCPID) field. If the RCE bit is cleared to ‘0’, then this field shall be cleared
to 00b.
Value Definition
00b A persistent event log reporting context does not already exist.
01b The reporting context was established by an NVM subsystem port.
10b The reporting context was established by a Management Endpoint
(refer to the NVM Express Management Interface Specification).
11b Reserved

15:00
Reporting Context Port Identifier (RCPID): If the RCE bit is cleared to ‘0’, then this
field shall be cleared to 0h.
If the RCE bit is set to ‘1’, then this field contains a Port Identifier of the type indicated
in the RCPIT field.
If the RCPIT field is set to 01b, then this field shall contain the Port Identifier  of the
NVM subsystem port that established the reporting context as defined in the Primary
Controller Capabilities data structure (refer to section 5.1.13.3.1).
If the RCPIT field is set to 10b, then:
• the least-significant byte of this field shall contain the Port Identifier of the
Management Endpoint  (refer to the NVM Express Management Interface
Specification); and
• the most-significant byte of this field shall be cleared to 0h.

479:378 Reserved
