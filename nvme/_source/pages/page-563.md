---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 563
headings: []
---

# Source page 563

NVM Express® Base Specification, Revision 2.1

541
specified in this specification are the number of random 4 KiB reads, the number of writes in Optimal Write
Size, and time in the Deterministic Window. Additional attributes are vendor specific.
For reads, writes, and time in the Deterministic Window, two values are provided in the Predictable Latency
Per NVM Set log page (refer to section 5.1.12.1.11):
• A typical or maximum amount of that attribute that the host may consume during any given DTWIN;
and
• A reliable estimate of the amount of that attribute that remains to be consumed during the current
DTWIN.
Figure 629 shows how the Typical, Maximum, and Reliable Estimates for the DTWIN attributes increase or
decrease when the associated NVM Set is in the Deterministic Window or Non -Deterministic Window.
Figure 629: DTWIN Attributes and Estimates

An NVM Set may transition autonomously to the NDWIN if, since entry to the current DTWIN:
a) the number of reads is greater than the value indicated in the DTWIN Reads Typical field;
b) the number of writes is greater than the value indicated in the DTWIN Writes Typical field;
c) the amount of time indicated in the DTWIN Time Maximum field has passed; or
d) a Deterministic Excursion occurs.
Figure 630 is an example that shows the relationship between the typical and reliable estimate values for
DTWIN Reads. DTWIN Reads Reliable Estimate begins near the DTWIN Reads Typical value at the start
of the current DTWIN at time 0. During the first time increment, the host reads x units, and the value of the
reliable estimate at time t2 is decremented by approximately x. During the second time increment, the host
reads a smaller amount consisting of y units and thus the reliable estimate at t3 is decremented by
approximately y.
