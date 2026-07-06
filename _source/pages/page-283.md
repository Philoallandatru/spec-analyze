---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 283
headings: ["5.1.12.1.19 NVMe-MI Commands Supported and Effects (Log Page Identifier 13h)"]
---

# Source page 283

NVM Express® Base Specification, Revision 2.1

261
Figure 262: FID Supported and Effects Data Structure
Bits Description
04
Controller Capability Change (CCC): If this bit is set to ‘1’, then changing the attributes of the feature
identified by this FID  may change controller capabilities.  Controller capability changes include a
firmware update that changes the capabilities reported in the CAP property.
If this bit is cleared to ‘0’, then changing the attributes of the feature identified by this FID  does not
modify controller capabilities.
03
Namespace Inventory Change (NIC): If this bit is set to ‘1’, then changing the attributes of the feature
identified by this FID may change the number of namespaces or capabilities for multiple namespaces.
Namespace inventory changes include adding or removing namespaces.
If this bit is cleared to ‘0’, then changing the attributes of the feature identified by this FID  does not
modify the number of namespaces or capabilities for multiple namespaces.
02
Namespace Capability Change (NCC): If this bit is set to ‘1’, then changing the attributes of the
feature identified by this FID  may change the capabilities of a single namespace.  Namespace
capability changes include a logical format change.
If this bit is cleared to ‘0’, then changing the attributes of the feature identified by this FID  does not
modify any namespace capabilities for the specified namespace.
01
User Data Content Change (UDCC): If this bit is set to ‘1’, then changing the attributes of the feature
identified by this FID may modify user data content in one or more namespaces.
If this bit is cleared to ‘0’, then changing the attributes of the feature identified by this FID does not
modify user data content in any namespace.
00
FID Supported (FSUPP): If this bit is set to ‘1’, then this FID is supported by the controller.
If this bit is cleared to ‘0’, then this FID is not supported by the controller and all other fields in this
structure shall be cleared to 0h.
Refer to section 3.1.3.6 for the FID support requirements for each controller type.
5.1.12.1.19 NVMe-MI Commands Supported and Effects (Log Page Identifier 13h)
This log page describes the Management Interface Command Set commands (refer to the NVM Express
Management Interface Specification) that the controller supports using the NVMe -MI Send and NVMe -MI
Receive commands and the effects of those Management Interface Command Set commands on the state
of the NVM subsystem. The log page is defined in Figure 263.
Figure 263: NVMe-MI Commands Supported and Effects Log Page
Bytes Description
03:00
Management Interface Command Supported 0  (MICS0): Contains the NVMe -MI Commands
Supported and Effects data structure (refer to Figure 264) for the Management Interface command
with an opcode value of 0h.
07:04
Management Interface Command Supported 1  (MICS1): Contains the NVMe -MI Commands
Supported and Effects data structure (refer to Figure 264) for the Management Interface command
with an opcode value of 1h.
… …
1019:1016
Management Interface Command Supported 254  (MICS254): Contains the NVMe -MI
Commands Supported and Effects data structure (refer to Figure 264) for the Management
Interface command with an opcode value of 254.
1023:1020
Management Interface Command Supported 255  (MICS255): Contains the NVMe -MI
Commands Supported and Effects data structure (refer to Figure 264) for the Management
Interface command with an opcode value of 255.
4095:1024 Reserved
The NVMe-MI Commands Supported and Effects data structure describes the overall possible effect of a
Management Interface command using the using the NVMe -MI Send command, including any optional
features of the command.
