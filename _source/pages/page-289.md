---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 289
headings: ["5.1.12.1.24 Management Address List (Log Page Identifier 18h)"]
---

# Source page 289

NVM Express® Base Specification, Revision 2.1

267
Figure 270: Dispersed Namespace Participating NVM Subsystems Log Page
Bytes Description
511:256 Participating NVM Subsystem Entry 0: This field contains the NQN of the NVM subsystem
containing the controller processing the command.
767:512
Participating NVM Subsystem Entry 1: This field contains the NQN of a participating NVM
subsystem that contains controllers that are able to provide access to the dispersed
namespace.
… …
(((N + 2) * 256) – 1):
((N + 1) * 256)
Participating NVM Subsystem Entry N: This field contains the NQN of a participating NVM
subsystem that contains controllers that are able to provide access to the dispersed
namespace.
5.1.12.1.24 Management Address List (Log Page Identifier 18h)
This log consists of a Management Address List (refer to Figure 271) of up to eight Management Address
Descriptors (refer to Figure 272). Management addresses are described in section 8.1.14.
Figure 271: Management Address List – Log Page
Bytes Description
511:00
Management Address Descriptor 0  (MAD0): This field contains the first Management Address
Descriptor (refer to Figure 272), if any. If the Management Address Type field is set to FFh, then
subsequent Management Address Descriptors are reserved.
1023:512
Management Address Descriptor 1 (MAD1): This field contains the second Management Address
Descriptor (refer to Figure 272), if any. If the Management Address Type field is set to FFh, then
subsequent Management Address Descriptors are reserved.
… …
4095:3584 Management Address Descriptor 7  (MAD7): This field contains the last Management Address
Descriptor (refer to Figure 272), if any.
Figure 272 describes the Management Address Descriptor.
Figure 272: Management Address Descriptor
Bytes Description
00
Management Address Type (MAT):  This field describes the contents of the Management Address
field:
Value Definition
0h The Management Address field is reserved.
1h The Management Address field contains the address of a management agent in the
NVM subsystem (e.g., in an Ethernet-attached SSD).
2h
The Management Address field contains the address of a fabric interface manager
(e.g., a Service URI for the Simple Network Management Protocol (SNMPv3) agent
as defined in RFC 4088).
FFh The Management Address field is reserved and all subsequent Management
Address Descriptors, if any, are reserved.
Other values Reserved

03:01 Reserved
511:04
Management Address (MADRS): A uniform resource indicator (URI; refer to RFC 3986) as a UTF -8
null-terminated string, containing the address of a management entity of the type indicated by the MAT
field.
Because the list is able to contain from zero to eight Management Address Descriptors with a MAT field
less than FFh, a host scans the list starting with Management Address Descriptor 0, and ending with either
Management Address Descriptor 7 or a Management Address Descriptor having a MAT field set to FFh.
