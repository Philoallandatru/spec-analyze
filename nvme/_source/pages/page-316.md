---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 316
headings: ["5.1.12.3.4 Pull Model DDC Request Log Page (Log Page Identifier 73h)"]
---

# Source page 316

NVM Express® Base Specification, Revision 2.1

294
The format of the AVE Discovery Log Page Entry is shown in Figure 302.
Figure 302: Get Log Page – AVE Discovery Log Page Entry
Bytes Description
Header
03:00 Total Entry Length (TEL):  This field indicates the length in bytes of the AVE Discovery
Log Page Entry.
227:04 AVE NQN (AVENQN):  This field indicates the NQN (represented as a null -terminated
string, NULL padded as necessary to the 224-byte maximum length) of the AVE.
228 Number of AVE Transport Records (N UMATR): This field indicates the number of
subsequent AVE transport records.
231:229 Reserved
AVE Transport Record List
251:232 AVE Transport Record 1: The first AVE Transport Record (refer to Figure 303), if any.
271:252 AVE Transport Record 2 : The second AVE Transport Record (refer to Figure 303), if
any.
… …
(NUMATR*20)+231:
(NUMATR*20)+212
AVE Transport Record NUMATR: The last AVE Transport Record (refer to Figure
303), if any.
The format of the AVE Transport Record is shown in Figure 303.
Figure 303: AVE Transport Record
Bytes Description
00
AVE Address Family (AVEADRFAM): This field identifies the IP address family. This field shall be set
to one of the following values:
• 1h: IPv4 address family; or
• 2h: IPv6 address family.
01 Reserved
03:02 AVE Transport Service Identifier (AVETRSVCID): This field identifies the TCP port.
19:04
AVE Transport Address (AVETRADDR): This field identifies the IP address. An IPv6 address is
encoded in binary as follows:
Bytes Description
15:00 IPv6 Address (IPV6ADDR): This field contains an IPv6 address.
An IPv4 address is encoded in binary as follows:
Bytes Description
03:00 IPv4 Address (IPV4ADDR): This field contains an IPv4 address.
15:04 Reserved

5.1.12.3.4 Pull Model DDC Request Log Page (Log Page Identifier 73h)
The format of the Pull Model DDC Request log page is shown in Figure 304.
