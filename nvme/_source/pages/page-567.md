---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 567
headings: ["8.1.19.2 Reachability event notifications"]
---

# Source page 567

NVM Express® Base Specification, Revision 2.1

545
o that NSID 30 and NSID 31 are able to reach each other;
• Reachability Association 3 (i.e., RA 3):
o specifies that:
▪ NSID 11 and NSID 30 are able to reach each other; and
▪ NSID 11 and NSID 31 are able to reach each other;
and
o does not specify any reachability between NSID 30 and NSID 31;
• The lack of a Reachability Association that contains Reachability Group B (i.e., RG B) and
Reachability Group D (i.e., RG D) specifies:
o that NSID 10 and NSID 11 are not able to reach each other;
and
• The lack of a self -referencing Reachability Association for Reachability Group E (i.e. , RG E)
specifies:
o that NSID 22 and NSID 23 are not able to reach each other.
Figure 631: Reachability Associations Example
RA 1
RGA
RGB
RA 2
RGA
RA 3
RGA
RGD
RA 4
RGB
RGC
RA 5
RGC
RGD
RA 6
RGD
RGE
NVM Subsystem
RG A
NVM
NSID 30
NVM
NSID 31
RG B
Memory
NSID 10
RG C
Compute
NSID 20
RG D RG E
Compute
NSID 22
Compute
NSID 23
Memory
NSID 11

 Reachability event notifications
Reachability may generate a Reachability Group Change notice (refer to section 5.1.2.1) or a Reachability
Association Change notice (refer to section 5.1.2.1).
Receipt of a Reachability Group Change Notice from a controller may indicate:
a) an NSID has been added to one or more of the Reachability Group Descriptors;
b) an NSID has been removed from one or more of the Reachability Group Descriptors;
c) a Reachability Group no longer has any NSIDs attached to this controller as members of that
Reachability Group and therefore is no longer reported in the Reachability Groups log page for this
controller; or
d) the NSID of a namespace has moved from one Reachability Group Descriptor to a different
Reachability Group Descriptor (i.e., the RGRPID field in the I/O Command Set Independent Identify
