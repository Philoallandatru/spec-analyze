---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 71
headings: ["3.1.4 Controller Properties"]
---

# Source page 71

NVM Express® Base Specification, Revision 2.1

49
Figure 32: Feature Support Requirements
Feature Name Feature
Identifier
Controller Support Requirements 1 Reference
I/O Administrative Discovery
Namespace Write Protection Config 84h O O P 5.1.25.1.31
Boot Partition Write Protection Config 85h O O P 5.1.25.1.32
Notes:
1. O/M/P definition: O = Optional, M = Mandatory, P = Prohibited
2. Mandatory for NVMe over PCIe. This feature is not supported for NVMe over Fabrics.
3. Mandatory if reservations are supported as indicated in the Identify Controller data structure.
4. Mandatory if reservations are supported by the namespace as indicated by a non-zero value in the Reservation
Capabilities (RESCAP) field in the Identify Namespace data structure.
5. Optional for NVM subsystems that do not implement a Management Endpoint. For NVM subsystems that
implement any Management Endpoint refer to the NVM Express Management Interface Specification.
6. Mandatory for controllers that support the Flexible Data Placement capability (refer to section 8.1.10). Refer to
the FDPS bit in Figure 312.
7. Optional if not required by the NVMe Transport (refer to section 3.9).
8. Mandatory if Telemetry Log, Firmware Commit or SMART/Health Critical Warnings are supported.
9. Prohibited for an Exported NVM Subsystem (refer to section 8.3.3).
10. For Discovery controllers that support explicit persistent connections, this feature is mandatory. For Discovery
controllers that do not support explicit persistent connections, this feature is prohibited.
 Controller Properties
A property is a dword, or qword attribute of a controller. The attribute may have read, write, or read/write
access. The host shall access a property using the width specified for that property with an offset that is at
the beginning of the property unless otherwise noted in a transport specific specification . All reserved
properties and all reserved bits within properties are read-only and return 0h when read.
For message-based controllers, properties may be read with the Property Get command and may be written
with the Property Set command.
For memory-based controllers, refer to the applicable NVMe Transport binding specification for access
methods and rules (e.g., NVMe PCIe Transport Specification).
Figure 33 describes the property map for the controller.
Accesses that target any portion of two or more properties are not supported.
Software should not rely on 0h being returned.
Figure 33: Property Definition
Offset
(OFST)
Size
(in
bytes)
I/O
Controller1
Administrative
Controller1
Discovery
Controller1 Name
0h 8 M M M CAP: Controller Capabilities
8h 4 M M M VS: Version
Ch 4 M2 M2 R INTMS: Interrupt Mask Set
Fh 4 M2 M2 R INTMC: Interrupt Mask Clear
14h 4 M M M CC: Controller Configuration
18h  R R R Reserved
1Ch 4 M M M CSTS: Controller Status
20h 4 O O R NSSR: NVM Subsystem Reset
24h 4 M2 M2 R AQA: Admin Queue Attributes
28h 8 M2 M2 R ASQ: Admin Submission Queue Base
Address
