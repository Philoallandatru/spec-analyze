---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 419
headings: []
---

# Source page 419

NVM Express® Base Specification, Revision 2.1

397
Figure 431: Set Features – Command Dword 11
Bits Description
14:13
Element Action (EA): This field specifies the action to perform on the specified Host Metadata Feature
value for each Metadata Element Descriptor data structure contained in the Host Metadata data
structure.
Value Definition
00b Add or Replace Entry
01b Delete Entry Multiple
10b Add Entry Multiple
11b Reserved
If the Element Action field is cleared to 00b (Add/Replace Entry) and the Metadata Element Descriptor
with the specified Element Type (refer to  Figure 433) does not exist in the specified Host Metadata
Feature value, then the Controller shall create the descriptor in the specified Host Metadata Feature
value with the value in the Host Metadata data structure.
If the Element Action field is cleared to 00b (Add/Replace Entry) and one Metadata Element Descriptor
with the specified Element Type exists in the specified Host Metadata Feature value, then the Controller
shall replace with the value in the specified Host Metadata data structure.
If the Element Action field is cleared to 00b (Add/Replace Entry) and the Feature Identifier field is set to
Enhanced Controller Metadata, then the controller shall abort the Set Features command with a status
code of Invalid Field in Command and shall not change any Host Metadata Feature value.
If the Element Action field is set to 01b (Delete Entry Multiple), then the Controller shall delete all the
specified Metadata Element Descriptors from the specified Host Metadata Feature value, if any. If none
of the specified Metadata Element Descriptors are present in the specified Host Metadata Feature value,
then the controller shall complete the Set Features command with a status code of Successful
Completion and shall not change any Host Metadata Feature value.
If the Element Action field is set to 10b (Add Entry Multiple), the Feature Identifier field is set to Enhanced
Controller Metadata, and no Metadata Element Descriptor with the specified Element Type exists in the
Enhanced Controller Metadata Feature value , then the controller shall create new Metadata Element
Descriptors in the Enhanced Controller Metadata Feature value with the Element Type and the value
specified in the Host Metadata data structure.
If the Element Action field is set to 10b (Add Entry Multiple), the Feature Identifier field is set to Enhanced
Controller Metadata, and one or more Metadata Element Descriptors with the specified Element Type
exists in the Enhanced Controller Metadata Fea ture value, then the controller shall add the specified
Metadata Element to the Enhanced Controller Metadata Feature value and shall not modify any existing
Metadata Element Descriptors.
If the Element Action field is set to 10b (Add Entry Multiple) and the Feature Identifier field is not set to
Enhanced Controller Metadata, then the controller shall abort the Set Features command with a status
code of Invalid Field in Command and shall not change the Host Metadata Feature value.
12:00 Reserved
Metadata Element Descriptors may be added, replaced, or deleted based on the action specified in the
Element Action field. Modification of the Host Metadata Feature value  shall be performed by the controller
in an atomic manner.
If a Set Features command is submitted for a Host Metadata Feature, a Host Metadata data structure,
defined in Figure 432, is transferred in the data buffer for the command. The Host Metadata data structure
is 4 KiB in size and contains zero or more Metadata Element Descriptors. If host software attempts to add
or replace a Metadata Element that causes the Host Metadata Feature value of the specified feature to
grow larger than 4 KiB, then the controller shall abort the command with a  status code of  Invalid Field in
Command.
If the host receives a Host Metadata data structure via the Get Features command, then all of the Metadata
Element Descriptors present for the specified feature are added to a Host Metadata data structure (refer to
