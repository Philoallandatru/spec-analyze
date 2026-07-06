---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 353
headings: ["5.1.13.2.2 Active Namespace ID list (CNS 02h)", "5.1.13.2.3 Namespace Identification Descriptor list (CNS 03h)"]
---

# Source page 353

NVM Express® Base Specification, Revision 2.1

331
Figure 313: Identify – Power State Descriptor Data Structure
Bits Description
15:00
Maximum Power (MP):  This field indicates the sustained maximum power consumed by the NVM
subsystem in this power state. The power in Watts is equal to the value in this field multiplied by the scale
specified in the Max Power Scale  bit. A value of 0h indicates Maximum Power is not reported.  Refer to
section 8.1.17.
Note: This value is intended to provide an approximate guideline for hosts to manage power versus
performance. Platform and form factor specifications may have additional power measurement and
reporting requirements that are outside the scope of this specification.
Figure 314 describes the time scales indicated by the EPFVTS field, the FQVTS field, and the EPFRTS
field (refer to Figure 313).
Figure 314: Time Scale Values
Value Definition
0h 1 microsecond
1h 10 microseconds
2h 100 microseconds
3h 1 millisecond
4h 10 milliseconds
5h 100 milliseconds
6h 1 second
7h 10 seconds
8h 100 seconds
9h 1,000 seconds
Ah 10,000 seconds
Bh 100,000 seconds
Ch 1,000,000 seconds
Dh Reserved
Eh Reserved
Fh Reserved
5.1.13.2.2 Active Namespace ID list (CNS 02h)
A list of 1,024 namespace IDs is returned to the host containing active NSIDs in increasing order that are
greater than the value specified in the Namespace Identifier (NSID) field of the command. The controller
shall abort the command with a status code  of Invalid Namespace or Format if  the NSID field is set to
FFFFFFFEh or FFFFFFFFh. The NSID field may be cleared to 0h to retrieve a Namespace List including
the namespace starting with NSID of 1h. The data structure returned is a Namespace List (refer to section
4.6.2).
5.1.13.2.3 Namespace Identification Descriptor list (CNS 03h)
A list of Namespace Identification Descriptor structures (refer to Figure 315) is returned to the host for the
specified namespace if the value in the Namespace Identifier (NSID) field is an active NSID. The controller
shall abort the command with a status code of Invalid Namespace or Format if the NSID field is set to
FFFFFFFEh or is set to FFFFFFFFh.Namespace Identification Descriptor structures consist of one or more
Namespace Identifiers (NID) of various types as indicated by the Namespace Identifier Type (NIDT) field
in each descriptor. Each NID is assigned to a namespace at namespace creation and remains fixed
throughout the life of that namespace. If the NSID field does not specify an active NSID, then refer to section
3.2.1.5 for the status code to return.  Textual representations related to Boot of NIDs may be found in the
NVM Express Boot Specification.
The contents of the Namespace Identification Descriptor list is preserved across namespace and controller
operations (e.g., Controller Level Reset, namespace format, etc.).
