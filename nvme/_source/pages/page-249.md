---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 249
headings: []
---

# Source page 249

NVM Express® Base Specification, Revision 2.1

227
The log page indicates typical values and reliable estimates for attributes associated with the Deterministic
Window and the Non-Deterministic Window of the specified NVM Set. The Typical, Maximum, and Minimum
values are static and worst-case values over the lifetime of the NVM subsystem.
After the controller successfully completes a read of this log page with Retain Asynchronous Event bit
cleared to ‘0’, then reported events are cleared to ‘0’ for the specified NVM Set and the field corresponding
to the specified NVM Set is cleared to ‘0’ in the Predictable Latency Event Aggregate log page.  Co-
ordination between two or more hosts is beyond the scope of this specification.
Figure 220: Predictable Latency Per NVM Set Log Page
Bytes Description
00
Status (STSNVMS): This field indicates the status of the specified NVM Set.
Bits Description
7:3 Reserved
2:0
Predictable Latency Mode Window (PLMW): This field indicates the window for
the NVM Set when Predictable Latency Mode is enabled.
Value Definition
000b Not used (Predictable Latency Mode not enabled)
001b Deterministic Window (DTWIN)
010b Non-Deterministic Window (NDWIN)
011b to 111b Reserved


01 Reserved
03:02
Event Type (ETYP): This field specifies the event(s) that occurred for the NVM Set indicated.
Multiple bits may be set  to ‘1’. All bits are cleared  to ‘0’ after the log page is read with Retain
Asynchronous Event bit cleared to ‘0’.
Bits Description
15 Deterministic Excursion Autonomous Transition (DEAT):  Autonomous transition
from DTWIN to NDWIN due to Deterministic Excursion
14 Max Value Exceeded Autonomous Transition (MVEAT):  Autonomous transition
from DTWIN to NDWIN due to typical or maximum value exceeded
13:03 Reserved
02 DTWIN Time Warning (DTTW)
01 DTWIN Writes Warning (DTWW)
00 DTWIN Reads Warning (DTRW)

31:04 Reserved
Typical, Maximum, and Minimum Values
39:32 DTWIN Reads Typical (DTWRT): Indicates the typical number of 4 KiB random reads that may
be performed in the Deterministic Window. Refer to section 8.1.18.
47:40 DTWIN Writes Typical (DTWWT): Indicates the typical number of writes in units of the Optimal
Write Size that may be performed in the Deterministic Window. Refer to section 8.1.18.
55:48
DTWIN Time Maximum  (DTWTM): Indicates the maximum time in milliseconds that the NVM
Set is able to remain in a Deterministic Window before entering a Non -Deterministic Window.
Refer to section 8.1.18.
63:56
NDWIN Time Minimum High  (NTWTMH): Indicates the minimum time in milliseconds that the
NVM Set needs to remain in the Non -Deterministic Window before entering a Deterministic
Window. This is the time necessary to prepare for remaining in the Deterministic Window for
DTWIN Time Maximum. Refer to section 8.1.18.
71:64
NDWIN Time Minimum Low  (NTWTML): Indicates the minimum time in milliseconds that the
NVM Set needs to remain in the Non -Deterministic Window before entering a Deterministic
Window. This is regardless of the amount of time spent in the previous Deterministic Window.
Refer to section 8.1.18.
127:72 Reserved
Reliable Estimates
135:128
DTWIN Reads Estimate (DTWRE): Indicates a reliable estimate of the number of 4 KiB random
reads remaining in the current Deterministic Window, if applicable. This value decrements from
DTWIN Reads Typical to 0h based on host read activity and operating conditions. Refer to section
8.1.18.1.
