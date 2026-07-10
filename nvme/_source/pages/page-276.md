---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 276
headings: ["5.1.12.1.16 Media Unit Status (Log Page Identifier 10h)"]
---

# Source page 276

NVM Express® Base Specification, Revision 2.1

254
5.1.12.1.16 Media Unit Status (Log Page Identifier 10h)
This log page is used to describe the configuration and wear of Media Units (refer to section 8.1.4). The log
page contains one Media Unit Status Descriptor for each Media Unit accessible by the specified domain.
Each Media Unit Status Descriptor (refer to Figure 255) indicates the configuration of the Media Unit (e.g.,
to which Endurance Group the Media Unit is assigned, to which NVM Set the Media Unit is assigned, to
which Channels the Media Unit is attached) and indications of wear (e.g., the Available Spare field and the
Percentage Used field). The indications of wear change as the Media Unit is written and read.
If the NVM subsystem supports multiple domains, then the controller reports the Media Unit Status log page
for the domain specified in the Log Specific Identifier field (refer to  Figure 253), if accessible . If the
information is not accessible, then the log page is not available (refer to section 8.1.1.10). If the Log Specific
Identifier field is cleared to 0h, then the specified domain is the domain containing the controller that is
processing the command.
Media Unit Identifier values (refer to Figure 255) begin with 0h and increase sequentially . If the NVM
subsystem supports multiple domains, then the Media Unit Identifier values are unique within the specified
domain. If the NVM subsystem does not support multiple domains, then the Media Unit Identifier values are
unique within the NVM subsystem.
Media Unit Status Descriptors are listed in ascending order by Media Unit Identifier.
Requirements for supporting the Media Unit Status log page are defined in section 8.1.4.
Figure 253: Domain Identifier – Log Specific Identifier
Bits Description
15:00
Domain Identifier (DID): This field specifies the identifier for the domain (refer to section 3.2.5) used for
this log page.
If the NVM subsystem does not support multiple domains, then this field is reserved.
If this field specifies a non-zero Domain Identifier that is not reported in the Domain List (refer to section
5.1.13.2.15), then the controller shall abort the command with a status code of Invalid Field in Command.

Figure 254: Media Unit Status Log Page
Bytes Description
Header
01:00
Number of Media Unit Status Descriptors (NMU): This field indicates the number of Media Unit Status
Descriptors in the log page. If this field is cleared to 0h, then no Media Unit Status Descriptors are
reported.
03:02 Number of Channels (CCHANS):  This field indicates the number of Channels accessible by the
controller. A value of 0h indicates that the number of Channels accessible by the controller is not reported.
05:04
Selected Configuration (SELC): This field indicates the Configuration Identifier selected by the most
recent successful completion of the Capacity Management command Select Capacity Configuration
operation. If a Select Capacity Configuration operation has not been completed, this field may indicate a
non-zero value (i.e., a configuration was selected by default). If a Configuration Identifier is not selected,
then this field shall be cleared to 0h.
15:06 Reserved
Media Unit Status Descriptor List
NOTE 1 Media Unit Status Descriptor 0: This field contains the first Media Unit Status Descriptor (refer to Figure
255), if any.
NOTE 1 Media Unit Status Descriptor 1: This field contains the second Media Unit Status Descriptor, if any.
…
NOTE 1 Media Unit Status Descriptor NMU-1: This field contains the last Media Unit Status Descriptor, if any.
Notes:
1. Media Unit Status Descriptors may be different lengths.
The Media Unit Status Descriptor is defined in Figure 255.
