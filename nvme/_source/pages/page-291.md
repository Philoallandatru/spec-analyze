---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 291
headings: ["5.1.12.1.26 Reachability Associations (Log Identifier 1Bh)"]
---

# Source page 291

NVM Express® Base Specification, Revision 2.1

269
Figure 274: Reachability Groups Log Page
Bytes Description
Reachability Group Descriptor List
n:16 Reachability Group Descriptor 0: The first Reachability Group Descriptor (refer to Figure 275), if any.
m:n+1 Reachability Group Descriptor 1: The second Reachability Group Descriptor (refer to Figure 275), if
any.
… …
x:y Reachability Group Descriptor NRGD-1: The last Reachability Group Descriptor (refer to Figure 275),
if any.
The format of the Reachability Group Descriptor is defined in Figure 275. Namespace Identifiers shall be
listed in ascending NSID order.
Figure 275: Reachability Group Descriptor format
Bytes Description
Header
03:00
Reachability Group ID (RGID):  The Reachability Group Identifier associated with
namespaces in the Reachability Group (refer to section 8.1.19) described by this Reachability
Group Descriptor.
07:04
Number of NSID Values (NNID):  This field indicates the number of Namespace Identifier
values in this Reachability Group Descriptor.
If the RGO bit is set to ‘1’, then this field shall be cleared to 0h.
15:08
Change Count  (CHNGC): This field contains a 64 -bit incrementing count, indicating an
identifier for the information contained in this Reachability Group Descriptor. A value of 0h
indicates that the controller does not report a Change Count for this Reachability Group
Descriptor. If a Change Count is reported, then the count starts at 1h following a Controller
Level Reset and is incremented each time the data in this Reachability Group Descriptor
changes. If the value of this field is FFFFFFFF_FFFFFFFFh, then the field shall be set to 1h
when incremented (i.e., rolls over to 1h).
If this field contains 0h, the host should examine this Reachability Group Descriptor for any
changes and not use this field as an indicator that a change has occurred.
31:16 Reserved
Namespace Identifier List
35:32 Namespace Identifier 0: The Namespace Identifier of the first attached namespace, if any,
that is a member of this Reachability Group.
39:36 Namespace Identifier 1: The Namespace Identifier of the second attached namespace, if
any, that is a member of this Reachability Group.
… …
(((NNID-1)*4) + 35):
(((NNID-1)*4) + 32)
Namespace Identifier NNID-1: The Namespace Identifier of the NNID attached namespace,
if any, that is a member of this Reachability Group.
5.1.12.1.26 Reachability Associations (Log Identifier 1Bh)
This log page consists of a header describing the log page and a list of descriptors containing the
Reachability Associations (refer to section 8.1.19), for the controller processing the Get Log Page command
requesting the Reachability Associations log page, that contain Reachability Groups (refer 5.1.12.1.25) that
all have the same reachability attributes.
If the Index Offset Supported bit is cleared to ‘0’ in the LID Support and Effects data structure for this log
page (refer to Figure 204), then:
• if the RAO bit is cleared to ‘0’ in Command Dword 10, then the LPOL field in Command Dword 12
and the LPOU field in Command Dword 13 of the Get Log Page command should be cleared to
0h.
