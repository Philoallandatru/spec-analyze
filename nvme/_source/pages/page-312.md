---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 312
headings: ["5.1.12.3.2 Host Discovery (Log Page Identifier 71h)"]
---

# Source page 312

NVM Express® Base Specification, Revision 2.1

290
Figure 296: Discovery Log Page
Bytes Description
Header
07:00
Generation Counter (GENCTR): Indicates the version of the discovery information,
starting at a value of 0h. For each change in the Discovery log page, this counter is
incremented by one. If the value of this field is FFFFFFFF_FFFFFFFFh, then the field shall
be cleared to 0h when incremented (i.e., rolls over to 0h).
15:08 Number of Records (NUMREC): Indicates the number of records contained in the log
page.
17:16
Record Format (RECFMT): Specifies the format of the Discovery log page. If a new format
is defined, this value is incremented by one. The format of the record specified in this
definition shall be 0h.
18
Discovery Log Page Flags (DLPF):  This field indicates additional information related to
the Discovery log page.
Bits Description
7:3 Reserved
2
All Subsystems (ALLSUB): If this bit is set to ‘1’, then the records contained
in the log page are for all NVM subsystem ports. If this bit is cleared to ‘0’, then
the records contained in the log page may or may not be for all NVM subsystem
ports.
1
Port Local (PORTLCL): If this bit is set to ‘1’, then the records contained in the
log page are only for NVM subsystem ports that are presented through the
same NVM subsystem port that is returning this log. If this bit is cleared to ‘0’,
then the records contained in the log page may be for NVM subsystem ports
that are presented through any NVM subsystem port on that NVM subsystem.
0
Extended (EXTEND): If this bit is set to ‘1’, then the records contained in the
log page may include Extended Discovery Log Page Entries. If this bit is
cleared to ‘0’, then the records contained in the log page do not include
Extended Discovery Log Page Entries.

19 Reserved
23:20
Total Discovery Log Page Length (TDLPL): This field indicates the length in bytes of the
entire Discovery log page. If the value in this field is cleared to 0h, then this field is not
reported.
1023:24 Reserved
Discovery Log Page Entry List
(TEL – 1) + 1024:
1024
Entry 0: Contains either the first Discovery Log Page Entry as defined in Figure 294 or the
first Extended Discovery Log Page Entry as defined in Figure 295. TEL is the size indicated
in the Total Entry Length (TEL) field of the Extended Discovery Log Page Entry. For
Discovery Log Page Entries, TEL shall be 1,024 bytes.
… …
TDLPL - 1:
TDLPL - TEL
Entry N: Contains either the Nth Discovery Log Page Entry as defined in Figure 294 (if
present) or the Nth Extended Discovery Log Page Entry as defined in Figure 295 (if
present). TEL is the size indicated in the Total Entry Length (TEL) field of the Extended
Discovery Log Page Entry and TDLPL is the size indicated in the Total Discovery Log Page
Length (TDLPL) field. For Discovery Log Page Entries, TEL shall be 1 ,024 bytes. If the
TDLPL field is reserved (i.e., this log page does not contain any Extended Discovery Log
Page Entries), then the byte position of the Nth Discovery Log Page Entry shall be: (((N +
2) * 1024) - 1):((N + 1) * 1024).
5.1.12.3.2 Host Discovery (Log Page Identifier 71h)
The Host Discovery log page shall only be supported by Discovery controllers. The Host Discovery log
page shall not be supported by controllers that expose namespaces for NVMe over PCIe or NVMe over
Fabrics. The Host Discovery log page provides an inventory of hosts that have registered discovery
information with the Discovery controller. A host or a Discovery controller m ay obtain the host inventory
from this log page. Depending on the parameters specified in a Get Log Page command, the Host Discovery
log page may contain entries for:
• all known hosts; or
