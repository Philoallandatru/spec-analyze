---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 318
headings: []
---

# Source page 318

NVM Express® Base Specification, Revision 2.1

296
Figure 307: Identify – Command Dword 10
Bits Description
31:16
Controller Identifier (CNTID): This field specifies the controller identifier used as part of some Identify
operations. Whether the CNTID field is used for a particular Identify operation is indicated in Figure 310.
If this field is not used as part of the Identify operation, then:
• host software shall clear this field to 0h  for backwards compatibility (0h is a valid controller
identifier); and
• the controller shall ignore this field.
Controllers that support the Namespace Management capability (refer to section 8.1.15) shall support
this field.
15:08 Reserved
07:00 Controller or Namespace Structure (CNS): This field specifies the information to be returned to the
host. Refer to Figure 310.

Figure 308: Identify – Command Dword 11
Bits Description
31:24
Command Set Identifier (CSI):  This field is CNS value specific. This field specifies the I/O Command
Set to be used by the command for CNS values that require a Command Set Identifier. Refer to Figure
310 for Identify command CNS values that use this field. This field shall be cleared to 0h for Identify
operations with CNS values that do not use this field.
Values for this field are defined by Figure 311.
23:16 Reserved
15:00 CNS Specific Identifier (CNSSID): This field is dependent on the specified CNS value and if not defined
by that CNS value, then this field is reserved.
If the controller supports selection of a UUID by the Identify command (refer to section 8.1.28), then
Command Dword 14 is used to specify a UUID Index value (refer to Figure 309).
Figure 309: Identify – Command Dword 14
Bits Description
31:07 Reserved
06:00 UUID Index (UIDX): Refer to Figure 658.
The data structure returned is based on the Controller or Namespace Structure (CNS) field as shown in
Figure 310. If there are fewer entries to return for the data structure indicated based on CNS value, then
the unused portion of the returned data is zero filled. If a controller does not support the specified CNS
value, then the controller shall abort the command with a status code of Invalid Field in Command.
When issuing the Identify command, if the specified namespace is not associated with an I/O Command
Set that supports the specified Identify CNS value (refer to Figure 310), then the controller shall abort the
command with a status code of Invalid I/O Command Set.
Note: The CNS field was specified as a one bit field in revision 1.0 and is a two bit field in revision 1.1. Host
software should only issue CNS values defined in revision 1.0 to controllers compliant with revision 1.0.
Host software should only issue CNS values defined in revision 1.1 to controllers compliant with revision
1.1. T he results of issuing other CNS values to controllers compliant with revision 1.0 or  revision 1.1,
respectively, are indeterminate.
The Identify Controller data structure, Identify Namespace data structure, and the I/O Command Set
specific Identify Namespace data structure include several unique identifiers. The format and layout of these
unique identifiers is described in section 4.7.1.
