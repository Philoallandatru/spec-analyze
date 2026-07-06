---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 251
headings: []
---

# Source page 251

NVM Express® Base Specification, Revision 2.1

229
• if the RGO bit is cleared to ‘0’ in Command Dword 10, then the LPOL field in Command Dword 12
and the LPOU field in Command Dword 13 of the Get Log Page command should be cleared to
0h.
If the Index Offset Supported bit is set to ‘1’ in the LID Supported and Effects data structure for this log page
(refer to Figure 204) and the OT bit is set to ‘1’, then an index value specified in the Log Page Offset Lower
(LPOL) field (refer to Figure 199) and the Log Page Offset Upper (LPOU) field (refer to Figure 200) in the
Get Log Page command shall be used to index to the header or an ANA Group Descriptor within the list of
ANA Group Descriptors as shown in Figure 223 (e.g., specifying an index offset of 0 returns this log page
starting at byte offset 0h, specifying an index offset of 1 returns this log page starting at the offset of ANA
Group Descriptor 0, specifying an index offset of 2 returns this log page starting at the offset of ANA Group
Descriptor 1, etc.). Due to the complexity of using a byte offset with a variable sized ANA Group Descriptor,
the host should use an index offset by setting the OT bit to ‘1’ in Command Dword 14 of the Get Log Page
command.
If the host performs multiple Get Log Page commands to read the ANA log page (e.g., using the LPOL field
or the LPOU field), the host should re -read the header of the log page and ensure that the Change Count
field in the Asymmetric Namespace Access log matches the original value read. If it does not match, then
the data captured is not consistent and the ANA log page should be re-read.
The Log Specific Parameter field in Command Dword 10 (refer to Figure 197) for this log page is defined
in Figure 222.
Figure 222: Asymmetric Namespace Access Log Specific Parameter Field
Bits Description
14:09 Reserved
08
Return Groups Only (RGO):  If this bit is set to ‘1’, then the controller shall return ANA Group
Descriptors with the Number of NSID Values field in each ANA Group Descriptor cleared to 0h (i.e.,
no Namespace Identifiers are returned). If this bit is cleared to ‘0’, then the controller shall return
ANA Group Descriptors that contain the Namespace Identifiers of attached namespaces that are
members of the ANA Group described by that ANA Group Descriptor and the Number of NSID
Values field set to the number of Namespace Identifier values in that ANA Group Descriptor.

Figure 223: Asymmetric Namespace Access Log Page
Bytes Description
Header
07:00
Change Count (CHGC): This field contains a 64-bit incrementing Asymmetric Namespace Access
log change count, indicating an identifier for this set of asymmetric namespace access information.
The count starts at 0h following a Controller Level Reset and is incremented each time the contents
of the log page change (e.g., not only if an Asymmetric Namespace Access Change Asynchronous
Event Notification is generated). If the value of this field is FFFFFFFF_FFFFFFFFh, then the field
shall be cleared to 0h when incremented (i.e., rolls over to 0h).
09:08
Number of ANA Group Descriptors  (NAGD): This field indicates the number of ANA Group
Descriptors available in the log page. The log page shall contain one ANA Group Descriptor for each
ANA Group that contains namespaces that are attached to the controller.
If, for an ANA Group, there are no namespaces attached to the controller processing the command,
then no ANA Group Descriptor is returned for that ANA Group (i.e., an ANA Group Descriptor is
returned only if that ANA Group contains namespaces that are attached to the controller processing
the command).
If no namespaces are attached to the controller, then the log page does not contain any ANA Group
Descriptors and this field is cleared to 0h.
15:10 Reserved
List of ANA Group Descriptors
n:16 ANA Group Descriptor 0: This if the first ANA Group Descriptor (refer to Figure 224), if any.
m:n+1 ANA Group Descriptor 1: This if the second ANA Group Descriptor (refer to Figure 224), if any.
… …
x:y ANA Group Descriptor NAGD-1: This if the last ANA Group Descriptor (refer to Figure 224), if any.
