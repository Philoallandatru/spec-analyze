---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 406
headings: ["5.1.25.1.13 Predictable Latency Mode Window (Feature Identifier 14h)"]
---

# Source page 406

NVM Express® Base Specification, Revision 2.1

384
Predictable Latency Mode. Upon successful completion of this command, the controller shall be in the Non-
Deterministic Window.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 403 are returned
in Dword 0 of the completion queue entry for that command and the Deterministic Threshold Configuration
data structure is returned.
Figure 402: Predictable Latency Mode Config – Command Dword 11
Bits Description
31:16 Reserved
15:00 NVM Set Identifier (NVMSETID): This field specifies the NVM Set to be modified.

Figure 403: Predictable Latency Mode Config – Command Dword 12
Bits Description
31:01 Reserved
00
Predictable Latency Enable (LPE): If this bit is set to ‘1’, then Predictable Latency Mode (refer to section
8.1.18) is enabled for the NVM Set specified. If this bit is cleared to ‘0’, then Predictable Latency Mode
is disabled for the NVM Set specified.
Predictable Latency Events (refer to section 5.1.12.1.12) are configured as described in Figure 404.
Figure 404: Predictable Latency Mode – Deterministic Threshold Configuration Data Structure
Bytes Description
01:00
Enable Event (ENEV): This field specifies whether an entry shall be added to the Predictable Latency
Event Aggregate log page for the associated event. If a bit is set to ‘1’, then an entry shall be added if
the specified event occurs. If a bit is cleared to ‘0’, then an entry shall not be added if the specified event
occurs.
Bits Description
15 Deterministic Excursion (DTE): Autonomous transition from DTWIN to NDWIN due to
Deterministic Excursion.
14 Exceeded Value (EXV): Autonomous transition from DTWIN to NDWIN due to typical or
maximum value exceeded.
13:03 Reserved
02 DTWIN Time Warning (DTTW)
01 DTWIN Writes Warning (DTWW)
00 DTWIN Reads Warning (DTRW)

31:02 Reserved
39:32
DTWIN Reads Threshold (DTRT): If the value of DTWIN Reads Estimate falls below this value and the
DTWIN Reads Warning is enabled, then the ‘DTWIN Reads Warning’ event is set in the Predictable
Latency Per NVM Set log page for the affected NVM Set.
47:40
DTWIN Writes Threshold (DTWT): If the value of DTWIN Writes Estimate falls below this value and the
DTWIN Writes Warning is enabled, then the ‘DTWIN Writes Warning’ event is set in the Predictable
Latency Per NVM Set log page for the affected NVM Set.
55:48
DTWIN Time Threshold  (DTTT): If the value of DTWIN Time Estimate falls below this value and the
DTWIN Time Warning is enabled, then the ‘DTWIN Time Warning’ event is set in the Predictable Latency
Per NVM Set log page for the affected NVM Set.
511:56 Reserved
5.1.25.1.13 Predictable Latency Mode Window (Feature Identifier 14h)
This Feature is used to set the window for the specified NVM Set and its associated namespaces if the
NVM Set is configured in Predictable Latency Mode (refer to section 8.1.18). The attributes are specified in
Command Dword 11 and Command Dword 12. If Predictable Latency Mode is not enabled, then the
controller shall abort the command with a status code of Invalid Field in Command.
