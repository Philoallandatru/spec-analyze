---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 420
headings: ["5.1.25.1.26.1  Enhanced Controller Metadata (Feature Identifier 7Dh)"]
---

# Source page 420

NVM Express® Base Specification, Revision 2.1

398
Figure 432) and returned in the data buffer for that command. The data buffer size is equal to the size of
the Host Metadata data structure that is 4 KiB in size.
Figure 432: Host Metadata Data Structure
Bytes Description
Header
00 Number of Metadata Element Descriptors (NMED): This field contains the number of Metadata
Element descriptors in the data structure.
01 Reserved
Metadata Element Descriptor List
x:02 Metadata Element Descriptor 0: This field contains the first Metadata Element descriptor or 0h
if there are no entries.
y:x+1 Metadata Element Descriptor 1: This field contains the second Metadata Element descriptor or
0h if there is only 1 entry.
… …
4095:z Metadata Element Descriptor N: This field contains the last Metadata Element descriptor or 0h
if there are less than N+1 entries where N is the value of the NMED field.
If the Feature Identifier field specifies Controller Metadata or Namespace Metadata, then the Host Metadata
data structure may contain at most one Metadata Element Descriptor of each Element Type. If the Feature
Identifier field specifies Enhanced Controll er Metadata, then a Host Metadata data structure may contain
more than one Metadata Element Descriptor of each Element Type. Each Metadata Element Descriptor
contains the data structure shown in Figure 433.
Figure 433: Metadata Element Descriptor
Bits Description
31 + (Element
Length*8) :32 Element Value (EVAL): This field specifies the value for the element.
31:16
Element Length (ELEN): This field specifies the length of the Element Value field in bytes. This
field shall be cleared to 0h when deleting an entry (i.e., the EA field is set to 01b in Command
Dword 11). This field should be non -zero when adding/updating an entry (i.e., the EA field is
cleared to 00b). If this field is cleared to 0h when adding/updating an entry, then the controller
behavior is undefined.
15:12 Reserved
11:08
Element Revision (ER): This field specifies the revision of this element value. Unless specified
otherwise elsewhere in this specification, all Metadata Element Descriptors shall clear this field
to 0h.
07:05 Reserved
04:00
Element Type (ET): This field specifies the type of metadata stored in the descriptor.
Value Definition
00h Reserved
01h to 17h
Element Types defined by this specification . Enhanced Controller
Metadata Element and Controller Metadata Element types are defined in
Figure 434. Namespace Metadata Element types are defined in Figure 435.
18h to 1Fh Vendor Specific

  Enhanced Controller Metadata (Feature Identifier 7Dh)
This Feature is used to store enhanced controller metadata about the host platform for later retrieval.
The metadata element types defined in Figure 434 are used by this Feature.
Figure 434: Controller Metadata Element Types
Value Definition
00h Reserved
01h Operating System Controller Name: The name of the controller in the operating system as a UTF-8 string.
