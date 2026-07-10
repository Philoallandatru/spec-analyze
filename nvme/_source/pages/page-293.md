---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 293
headings: ["5.1.12.1.27 Changed Allocated Namespace List (Log Identifier 1Ch)"]
---

# Source page 293

NVM Express® Base Specification, Revision 2.1

271
Figure 278: Reachability Association Descriptor format
Bytes Description
Header
03:00
Reachability Association ID (RASID):  The Reachability Association Identifier associated
with Reachability Groups in the Reachability Association (refer to section 8.1.19) described
by this Reachability Association Descriptor.
07:04
Number of RGID Values  (NRID): This field indicates the number of Reachability Group
Identifier values in this Reachability Association Descriptor.
If the RAO bit is set to ‘1’, then this field shall be cleared to 0h.
15:08
Change Count  (CHNGC): This field contains a 64 -bit incrementing count, indicating an
identifier for the information contained in this Reachability Association Descriptor. A value of
0h indicates that the controller does not report a Change Count for this Reachability
Association Descriptor. If a Change Count is reported, then the count starts at 1h following a
Controller Level Reset and is incremented each time the data in this Reachability Association
Descriptor changes. If the value of this field is FFFFFFFF_FFFFFFFFh, then t he field shall
be set to 1h when incremented (i.e., rolls over to 1h).
If this field contains 0h, the host should examine this Reachability Group Descriptor for any
changes and not use this field as an indicator that a change has occurred.
16
Reachability Association Characteristics  (RAC): This field indicates the Access
characteristics for all Reachability Groups in this Reachability Association Descriptor for
operations between the namespaces represented by the Reachability Groups.
Value Definition Reference
01h Reachable with no performance
characteristics specified 8.1.19
02h Fast copy operations are
supported
The Fast copy operations section in the
NVM Command Set Specification
03h Fast copy operations are not
supported
The Fast copy operations section in the
NVM Command Set Specification
All
other
values
Reserved

31:17 Reserved
Reachability Group Identifier List
35:32 Reachability Group Identifier 0 : The Reachability Group Identifier of the first Reachability
Group, if any, that is a member of this Reachability Association.
39:36 Reachability Group Identifier 1: The Reachability Group Identifier of the second
Reachability Group, if any, that is a member of this Reachability Association.
… …
(((NRID-1)*4)+35):
(((NRID-1)*4)+32)
Reachability Group Identifier NRID -1: The Reachability Group Identifier of the last
Reachability Group, if any, that is a member of this Reachability Association.
5.1.12.1.27 Changed Allocated Namespace List (Log Identifier 1Ch)
This log page is used to describe changes to allocated namespaces (refer to section 1.5.6), since the last
time that this log page was read, that:
a) have changed information in their Identify Namespace data structures (refer to section 1.5.49);
b) were previously unattached to the controller and have since been attached to the controller ;
c) were previously attached to the controller and have since been unattached from the controller ;
d) were created; and
e) were deleted.
The log page contains a Namespace List with up to 1,024 entries. If more than 1,024 namespaces have
changed attributes since the last time the log page was read, the first entry in the log page shall be set to
FFFFFFFFh and the remainder of the list shall be zero filled.
