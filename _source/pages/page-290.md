---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 290
headings: ["5.1.12.1.25 Reachability Groups (Log Identifier 1Ah)"]
---

# Source page 290

NVM Express® Base Specification, Revision 2.1

268
5.1.12.1.25 Reachability Groups (Log Identifier 1Ah)
This log page consists of a header describing the log page and a list of descriptors containing the
Reachability Groups (refer to section 8.1.19) that contain namespaces attached to the controller processing
this command that all have the same reachability attributes.
If the Index Offset Supported bit is cleared to ‘0’ in the LID Support and Effects data structure for this log
page (refer to Figure 204), then:
• if the RGO bit is cleared to ‘0’ in Command Dword 10, then the LPOL field in Command Dword 12
and the LPOU field in Command Dword 13 of the Get Log Page command should be cleared to
0h.
If the Index Offset Supported bit is set to ‘1’ in the LID Supported and Effects data structure for this log page
(refer to Figure 204), then:
• for indexes greater than 0, the entry data structure that is indexed is a Reachability Group descriptor
(e.g., specifying an index offset of 2 returns this log page starting at the second descriptor (i.e.,
Reachability Group Descriptor 1)); and
• for index 0, the data structure starting from the beginning of the log page is returned.
If the host performs multiple Get Log Page commands to read this log page (e.g., using the LPOL field or
the LPOU field), then the host should re-read the header of the log page and ensure that the Change Count
field in the log page matches the original value read. If it does not match, then the data captured is not
consistent and this log page should be re-read.
The Log Specific Parameter field in Command Dword 10 (refer to Figure 197) for this log page is defined
in Figure 273.
Figure 273: Reachability Groups Log Specific Parameter Field
Bits Description
14:09 Reserved
08
Return Groups Only (RGO): If this bit is set to ‘1’, then the controller shall return Reachability Group
Descriptors with the Number of NSID Values field in each Reachability Group Descriptor cleared to 0h
(i.e., no Namespace Identifiers are returned). If this bit is cleared to ‘0’, then the controller shall return
Reachability Group Descriptors that contain the Namespace Identifiers of attached namespaces that are
members of the Reachability Group described by that Reachability Group Descriptor and the Number of
NSID Values field set to the numbe r of Namespace Identifier values in that Reachability Group
Descriptor.

Figure 274: Reachability Groups Log Page
Bytes Description
Header
07:00
Change Count  (CHNGC): This field contains a 64 -bit incrementing Reachability Group s log page
change count, indicating an identifier for this set of reachability groups information. The count starts at
0h following a Controller Level Reset and is incremented each time the contents of the log page change.
If the value of this field is FFFF FFFF_FFFFFFFFh, then the field shall be set to 0h when incremented
(i.e., rolls over to 0h).
09:08
Number of Reachability Group Descriptors (NRGD): This field indicates the number of Reachability
Group Descriptors available in the log page. The log page shall contain one Reachability Group
Descriptor for each Reachability Group that contains namespaces that are attached to the controller.
If, for a Reachability Group, there are no namespaces attached to the controller processing the
command, then no Reachability Group Descriptor is returned for that Reachability Group (i.e., a
Reachability Group Descriptor is returned only if that Reachability Group contains namespaces that are
attached to the controller processing the command).
If no namespaces are attached to the controller, then the log page does not contain any Reachability
Group Descriptors and this field is cleared to 0h.
15:10 Reserved
