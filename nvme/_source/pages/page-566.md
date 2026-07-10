---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 566
headings: []
---

# Source page 566

NVM Express® Base Specification, Revision 2.1

544
• support Reachability Association Change Notices (refer to section 5.1.25.1.5);
• support the Reachability Groups log page; and
• support the Reachability Associations log page.
Namespaces attached to a controller in an NVM subsystem that supports Reachability Reporting shall:
• be members of a Reachability Group; and
• supply a valid Reachability Group Identifier in the Reachability Group Identifier (RGRPID) field in
the I/O Command Set Independent Identify Namespace Data Structure (refer to Figure 319).
Reachability and characteristics of reachability are indicated in the combination of the Reachability Groups
log page (refer to section 5.1.12.1.25) and the Reachability Associations log page (refer to section
5.1.12.1.26). A Reachability Group defines a group of namespaces that are all associated with namespaces
in other Reachability Groups defined by a Reachability Association. The method of assigning Reachability
Group identifiers and Reachability Association identifiers is outside the scope of this sp ecification. If
members of a Reachability Group are able to reach members of that Reachability Group, then a
Reachability Association that contains only that Reachability Group specifies the characteristics of that
reachability. All Reachability Groups in a Reachability Association have identical access characteristics. All
namespaces in a Reachability Group have identical access characteristics. If a Reachability Group is not
in any Reachability Association then the members of that Reachability Group are n ot reachable by any
other namespaces, including the members of that Reachability Group. A namespace is always able to reach
itself (i.e., a command that has two or more NSIDs as part of the command where those NSIDs are all the
same is not constrained by reachability).
If the Reachability Association Characteristics field (refer to Figure 278) is set to is 01h, then all namespaces
in the associated Reachability Groups are able to reach other namespaces in the other Reachability Groups
in that Reachability Association without any indication of a performance characteristic (e.g., for the Execute
command in the Computational Programs Command Set).  If the Reachability Association Characteristics
field is set to 02h, then all namespaces in the associated Reachability Groups are reachable and support
fast copy operations as defined by the applicable I/O Command Set specification (e.g., the Fast copy
operations section in the NVM Command Set Specification). If the Reachability Association Characteristics
field is set to 03h, then all namespaces in the associated Reachability Groups are reachable but do  not
support fast copy operations.
A namespace is only allowed to be in one Reachability Group. A Reachability Group is in zero or more
Reachability Associations with different values of the Reachability Association Characteristics field for each
Reachability Association.
The Reachability Group Identifier (RGRPID) for each Reachability Group shall be unique within the NVM
subsystem. If RGIDC bit in the CRCAP field in the Identify Controller data structure is set to ‘1’, then the
Reachability Group Identifier shall not change while the namespace is attached to any controller in the NVM
subsystem. If RGIDC bit in the CRCAP field is cleared to ‘0’, then the Reachability Group Identifier may
change while the namespace is attached to any controller in the NVM subsystem. If the Reachability Group
Identifier changes, the controller shall issue the Reachability Groups Change Notice as described in this
section.
Figure 631 is an example configuration that shows that:
• Reachability Association 1 (i.e., RA 1):
o specifies that:
▪ NSID 10 and NSID 30 are able to reach each other; and
▪ NSID 10 and NSID 31 are able to reach each other;
and
o does not specify any reachability between NSID 30 and NSID 31;
• Reachability Association 2 (i.e., RA 2) is a self-reference for Reachability Group A (i.e., RG A) that
specifies:
