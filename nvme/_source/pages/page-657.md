---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 657
headings: ["8.3.2.4 Asynchronous Events", "8.3.3 Exporting NVM Resources", "8.3.3.1 Management of Exported NVM Resources (Informative)"]
---

# Source page 657

NVM Express® Base Specification, Revision 2.1

635
Figure 717: Log Page Request Operation Specific Parameters for Pull Model DDC Request Log
Page
Bytes Description
00 Log Page Request Log Page Identifier (LPRLID)
01 Log Page Request Log Specific Parameter (LPRLSP)
03:02 Reserved
The LPRLID field and the LPRLSP field have respectively the same format and semantics of the LID field
and the LSP field in Command Dword 10 of a Get Log Page command. For example, to retrieve the Host
Discovery log page, the LPRLID field is set to 71h.
 Asynchronous Events
A Centralized Discovery controller (CDC) reports a Discovery Log Page Change Asynchronous Event
notification (Asynchronous Event Information field set to F0h) to each host that has requested
asynchronous event notifications of this type (refer to Figure 151) as specified in section 3.1.3.3.2 when a
Fabric Zoning configuration changes. In particular:
• if a Zone member with the Role set to 2h (i.e., an NVM subsystem) is added or removed from a
Zone, then the CDC shall report an AEN to all Zone members of that Zone having the Role set to
1h (i.e., all hosts in that Zone);
• if a Zone member with the Role set to 1h (i.e., a host) is added or removed from a Zone, then the
CDC shall report an AEN to all Zone members of that Zone having the Role set to 2h (i.e., all NVM
subsystems in that Zone); and
• If a Zone is added or removed, then the CDC shall report an AEN to all Zone members of the added
or removed Zone.
 Exporting NVM Resources
Exported Resource Management is a capability that may be supported in an NVM subsystem that only
contains controllers that use transports other than the Memory -Based Transport Model (e.g., the PCIe
transport).
Exporting an Underlying Namespace may be achieved by:
• creating an Exported NVM Subsystem (refer to section 5.3.2);
• assigning an Underlying Namespace to an Exported NVM Subsystem (refer to section 5.3.7.1.1);
• attaching a transport to the Exported NVM Subsystem to enable remote access to the Exported
NVM Subsystem (refer to section 5.3.9.1.1); and
• optionally assigning access control policies for the Exported NVM Subsystem (refer to sections
5.3.8.1.3 and 5.3.8.1.4).
Exported NVM Subsystems shall not expose information about Underlying NVM Subsystem resources or
entities that are not associated with Exported NVM Subsystem entities. This includes entities in Underlying
NVM Subsystems (e.g., NSID, NVM Set Identifiers, ANA Group Identifiers).
 Management of Exported NVM Resources (Informative)
Commands for managing Exported NVM Resources are processed by a controller in an Underlying NVM
Subsystem.
This section describes example flows to manage Exported NVM Resources through the use of the:
• Identify command with CNS set to 1Dh to retrieve the Underlying Namespace List;
• Identify command with CNS set to 1Eh to retrieve the list of Underlying Ports that may be used to
export NVMe over Fabrics Subsystems;
• Create Exported NVM Subsystem command to create an Exported NVM Subsystem;
• Manage Exported NVM Subsystem command to manage access to an Exported NVM Subsystem;
