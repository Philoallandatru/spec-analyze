---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 252
headings: ["5.1.12.1.14 Persistent Event (Log Page Identifier 0Dh)"]
---

# Source page 252

NVM Express® Base Specification, Revision 2.1

230
The format of the ANA Group Descriptor is defined in Figure 224. Namespace Identifiers shall be listed in
ascending NSID order.
Figure 224: ANA Group Descriptor format
Bytes Description
Header
03:00 ANA Group ID  (AGID): The ANA Group Identifier associated with all namespaces in the
ANA Group (refer to section 8.1.1.2) described by this ANA Group Descriptor.
07:04
Number of NSID Values  (NNV): This field indicates the number of Namespace Identifier
values in this ANA Group Descriptor.
If the RGO bit is set to ‘1’, then this field is cleared to 0h.
15:08
Change Count  (CHGC): This field contains a 64 -bit incrementing count, indicating an
identifier for the information contained in this ANA Group Descriptor. A value of 0h indicates
that the controller does not report a Change Count for this ANA Group Descriptor. If a Change
Count is reported, then the count starts at 1h following a Controller Level R eset and is
incremented each time the data in this ANA Group Descriptor change. If the value of this field
is FFFFFFFF_FFFFFFFFh, then the field shall be set to 1h when incremented (i.e., rolls over
to 1h).
If this field contains 0h, the host should examine this ANA Group Descriptor for any changes
and not use this field as an indicator that a change has occurred.
16
Asymmetric Namespace Access State Attributes (ANASA):
Bits Description
07:04 Reserved
03:00
Asymmetric Namespace Access State  (ANAS): This field indicates the
Asymmetric Namespace Access state for all namespaces in this ANA
Group when accessed through this controller.
Value Asymmetric Namespace Access State Reference
01h ANA Optimized state 8.1.1.4
02h ANA Non-Optimized state 8.1.1.5
03h ANA Inaccessible state 8.1.1.6
04h ANA Persistent Loss state 8.1.1.7
0Fh ANA Change state 8.1.1.8
All other
values Reserved


31:17 Reserved
Namespace Identifier List
35:32 Namespace Identifier 0: The Namespace Identifier of the first namespace that is a member
of this ANA Group.
39:36 Namespace Identifier 1: The Namespace Identifier of the second namespace, if any, that
is a member of this ANA Group.
… …
(((NNV-1)*4) + 35):
(((NNV-1)*4) + 32)
Namespace Identifier NNV-1: The Namespace Identifier of the last namespace that is a
member of this ANA Group.
5.1.12.1.14 Persistent Event (Log Page Identifier 0Dh)
The Persistent Event log page contains information about significant events not specific to a particular
command. The information in this log page shall be retained across power cycles and resets. NVM
subsystems should be designed for minimal loss of event information upon power failure. This log page
consists of a header describing the log and zero or more Persistent Events (refer to section 5.1.12.1.14.1).
This log page is global to the NVM subsystem.
The Log Specific Parameter field in Command Dword 10 (refer to Figure 197) for this log page is defined
in Figure 225.
