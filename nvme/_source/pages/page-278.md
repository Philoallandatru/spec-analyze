---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 278
headings: ["5.1.12.1.17 Supported Capacity Configuration List (Log Page Identifier 11h)"]
---

# Source page 278

NVM Express® Base Specification, Revision 2.1

256
Figure 255: Media Unit Status Descriptor
Bytes Description
Channel Identifier List
CIO+1 : CIO Channel 0 Identifier: This field contains the identifier for the first Channel attached to this
Media Unit, if any.
CIO+3 : CIO+2 Channel 1 Identifier: This field contains the identifier for the second Channel attached to this
Media Unit, if reported, if any.
… …
(MUCS*2)+CIO-1 :
(MUCS*2)+CIO-2
Channel MUCS-1 Identifier: This field contains the identifier for the last Channel attached to
this Media Unit, if any.
5.1.12.1.17 Supported Capacity Configuration List (Log Page Identifier 11h)
This log page is used to provide a list of Supported Capacity Configuration Descriptors (refer to Figure 256).
Each entry in the list defines a different configuration of Endurance Groups supported by the domain
specified in the Log Specific Identifier field in Command Dword 11 of the Get Log Page command as defined
in Figure 253.
If the NVM subsystem supports multiple domains, then the controller reports the Supported Capacity
Configuration List log page for the domain specified in the Log Specific Identifier field (refer to Figure 198),
if accessible. If the information is not accessible, then the log page is not available (refer to section 8.1.1.3).
If the Log Specific Identifier field is cleared to 0h, then the specified domain is the domain containing the
controller that is processing the command.
If the NVM subsystem supports multiple domains, then Capacity Configuration Identifier values are unique
within the specified domain. If the NVM subsystem does not support multiple domains, then Capacity
Configuration Identifier values are unique within the NVM subsystem.
In the Supported Capacity Configuration List (refer to Figure 256), Capacity Configuration Descriptors shall
be listed in ascending order by Capacity Configuration Identifier, and each Capacity Configuration Identifier
shall appear only once.
Figure 256: Supported Capacity Configuration List Log Page
Bytes Description
Header
0
Number of Supported Capacity Configurations (SCCN):  This field indicates the number of
Capacity Configuration Descriptors in the list. If this field is cleared to 0h, then no Capacity
Configuration Descriptors are reported.
15:1 Reserved
Capacity Configuration Descriptor List
NOTE 1 Capacity Configuration 0 Descriptor:  This field indicates the first Capacity Configuration
Descriptor (refer to Figure 257) in the list, if any.
NOTE 1 Capacity Configuration 1 Descriptor:  This field indicates the second Capacity Configuration
Descriptor in the list, if any.
…
NOTE 1 Capacity Configuration SCCN-1 Descriptor: This field indicates the last Capacity Configuration
Descriptor in the list, if any.
Notes:
1. Capacity Configuration Descriptors may be different lengths.
The Capacity Configuration Descriptor (refer to Figure 257) indicates the details of a particular configuration
of Endurance Groups and contains one Endurance Group Configuration Descriptor for each Endurance
Group accessible by the controller processing the command.
In the Capacity Configuration Descriptor (refer to Figure 257), Endurance Group Configuration Descriptors
shall be listed in ascending order by Endurance Group Identifier, and each Endurance Group Identifier shall
appear only once.
