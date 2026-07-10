---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 418
headings: []
---

# Source page 418

NVM Express® Base Specification, Revision 2.1

396
Figure 430: Get Features – Command Dword 11
Bits Description
00
Generate Default Host Metadata (GDHM): If this bit is set to ‘1’, then the controller shall modify the default
value of the specified Host Metadata feature by creating and returning  a number of vendor specific strings
for the Element Types of the specified Host Metadata feature. The number of vendor specific strings created
and returned is implementation specific. The controller is allowed to return:
• no vendor specific strings;
• vendor specific strings created for a subset of the defined Element Types of the specified Host
Metadata feature; or
• vendor specific strings created for each of the defined Element Types of the specified Host
Metadata feature.
These vendor specific strings replace the Metadata Element Descriptors in the default Host Metadata data
structure returned by the specified Host Metadata feature, when a Get Features command for a Host
Metadata feature with the SEL field set to 001b (i.e. , Default) is submitted, until a Controller Level Reset
occurs (i.e., the replacement default values are not persistent across a Controller Level Reset).
If this bit is cleared to ‘0’, then the controller shall not modify the default value of the specified Host Metadata
feature.
If this bit is cleared to ‘0’ and a Get Features command for a Host Metadata feature with the SEL field set to
001b (i.e., Default) is submitted, then the controller shall return the currently existing modified default value,
if any, for that Host Metadata feature (i.e., the updated default value that was created by the last Get
Features command, with the GDHM bit set to ‘1’, that completed successfully since the last Controller Level
Reset).
The host issues a Set Features command specifying one of the Host Metadata features containing a Host
Metadata data structure (refer to Figure 432). The host receives a Host Metadata data structure via the Get
Features command. The content of the strings in the Host Metadata data structure are vendor specific.
If any Get Features command specifying the GDHM bit set to ‘1’ returned a status code of Successful
Completion since the last Controller Level Reset, then for any subsequent Get Features command that
specifies a SEL field set to 001b (i.e., Default) and specifies a Host Metadata feature, the controller shall
return the replaced default value containing the most recent vendor specific strings for that Host Metada ta
feature.
The Action is specified in Command Dword 11 as shown in Figure 431.
Figure 431: Set Features – Command Dword 11
Bits Description
31:15 Reserved
