---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 292
headings: []
---

# Source page 292

NVM Express® Base Specification, Revision 2.1

270
If the Index Offset Supported bit is set to ‘1’ in the LID Supported and Effects data structure for this log page
(refer to Figure 204), then:
• for indexes greater than 0, the entry data structure that is indexed is a Reachability Association
descriptor (e.g., specifying an index offset of 2 returns this log page starting at the second descriptor
(i.e., Reachability Association Descriptor 1)).
If the host performs multiple Get Log Page commands to read this log page (e.g., using the LPOL field or
the LPOU field), then the host should re-read the header of the log page and ensure that the Change Count
field in this log page matches the original value read. If it does not match, then the data captured is not
consistent and this log page should be re-read.
The Log Specific Parameter field in Command Dword 10 (refer to Figure 197) for this log page is defined
in Figure 276.
Figure 276: Reachability Associations Log Specific Parameter Field
Bits Description
14:09 Reserved
08
Return Associations Only (RAO):  If this bit is set to ‘1’, then the controller shall return
Reachability Association Descriptors with the Number of RGID Values field in each Reachability
Association Descriptor cleared to 0h (i.e., no Reachability Group Identifiers are returned). If this
bit is cleared to ‘0’, then the controller shall return Reachability Association Descriptors that
contain the RGIDs of reachability groups that are members of the Reachability Association
described by that Reachability Association Descriptor and the Number of RGI D Values field set
to the number of Reachability Group Identifier values in that Reachability Association Descriptor.

Figure 277: Reachability Associations Log Page
Bytes Description
Header
07:00
Change Count (CHNGC): This field contains a 64-bit incrementing Reachability Associations log
change count, indicating an identifier for this set of reachability associations information. The
count starts at 0h following a Controller Level Reset and is incremented each time th e contents
of the log page change. If the value of this field is FFFFFFFF_FFFFFFFFh, then the field shall be
set to 0h when incremented (i.e., rolls over to 0h).
09:08
Number of Reachability Association Descriptors (NRAD):  This field indicates the number of
Reachability Association Descriptors available in the log page. The log page shall contain one
Reachability Association Descriptor for each Reachability Association that contains a
Reachability Group that contains namespaces that are attached to the controller.
If, for a Reachability Association, there are no Reachability Groups that contains namespaces
that are attached to the controller processing the command, then no Reachability Association
Descriptor shall be returned for that Reachability Association (i.e.,  a Reachability Association
Descriptor is returned only if that Reachability Association contains Reachability Groups
associated with the controller processing the command.
If no namespaces are attached to the controller, then the log page does not contain any
Reachability Association Descriptors and this field shall be cleared to 0h.
15:10 Reserved
Reachability Association Descriptor List
n:16 Reachability Association Descriptor 0 : This is the first Reachability Association Descriptor
(Figure 278), if any.
m:n+1 Reachability Association Descriptor 1: This is the second Reachability Association Descriptor
(Figure 278), if any.
… …
x:y Reachability Association Descriptor NRAD -1: This is the last Reachability Association
Descriptor (Figure 278), if any.
The format of the Reachability Association Descriptor is defined in Figure 278. Reachability Group
Identifiers shall be listed in ascending order.
