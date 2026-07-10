---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 315
headings: ["5.1.12.3.3 AVE Discovery Log Page (Log Page Identifier 72h)"]
---

# Source page 315

NVM Express® Base Specification, Revision 2.1

293
Figure 300: Host Discovery Log Page
Bytes Description
17:16
Record Format (RECFMT): This field indicates the format of the Host Discovery log page.
If a new format is defined, this value is incremented by one. The format of the record
specified in this definition shall be 0h.
18
Host Discovery Log Page Flags (HDLPF) : This field indicates additional information
related to the Host Discovery log page.
Bits Description
7:1 Reserved
0
All Hosts (ALLHOST): If this bit is set to ‘1’, then the records contained in the
log page are for all hosts. If this bit is cleared to ‘0’, then the records contained
in the log page may or may not be for all hosts.

19 Reserved
23:20 Total Host Discovery Log Page Length (THDLPL): This field indicates the length in bytes
of the entire Host Discovery log page.
1023:24 Reserved
Host Extended Discovery Log Page Entry List
(TEL - 1) + 1024:
1024
Host Extended Discovery Log Page Entry 0: This field contains the first Host Extended
Discovery Log Page Entry as defined in Figure 299. TEL is the size indicated in the Total
Entry Length (TEL) field of the Host Extended Discovery Log Page Entry.
… …
THDLPL - 1:
THDLPL – TEL
Host Extended Discovery Log Page Entry N: This field contains the Nth Host Extended
Discovery Log Page Entry as defined in Figure 299 (if present). TEL is the size indicated in
the Total Entry Length (TEL) field of the Host Extended Discovery Log Page Entry and
THDLPL is the size indicated in the Total Host Discovery Log Page Length (THDLPL) field.
5.1.12.3.3 AVE Discovery Log Page (Log Page Identifier 72h)
The format of the AVE Discovery Log Page is shown in Figure 301.
Figure 301: Get Log Page – AVE Discovery Log Page
Bytes Description
Header
07:00
Generation Counter (GENCTR):  This field indicates the version of the discovery
information, starting at a value of 0h. For each change in the AVE Discovery log page,
this field shall be incremented by one. If the value of this field is
FFFFFFFF_FFFFFFFFh, then the field shall be clea red to 0h when incremented (i.e.,
rolls over to 0h).
15:08 Number of Records (NUMREC): Indicates the number of records contained in the log
page.
17:16
Record Format (RECFMT):  This field indicates the format of the AVE Discovery log
page. If a new format is defined, this value is incremented by one. The format of the
record specified in this definition shall be 0h.
19:18 Reserved
23:20 Total AVE Discovery Log Page Length (TADLPL): This field indicates the length in
bytes of the entire AVE Discovery log page.
1023:24 Reserved
AVE Discovery Log Page Entry List
TEL + 1023:1024
AVE Discovery Log Page Entry 0: This field contains the first AVE Discovery Log
Page Entry as defined in Figure 302. TEL is the size indicated in the Total Entry Length
(TEL) field of the AVE Discovery Log Page Entry.
… …
TADLPL - 1:
TADLPL – TEL
AVE Discovery Log Page Entry NUMREC-1: This field contains the NUMREC-1 AVE
Discovery Log Page Entry as defined in Figure 302 (if present). TEL is the size indicated
in the Total Entry Length (TEL) field of the AVE Discovery Log Page Entry and TADLPL
is the size indicated in the Total AVE Log Page Length (TADLPL) field.
