---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 245
headings: []
---

# Source page 245

NVM Express® Base Specification, Revision 2.1

223
• If the DA4S bit is cleared to ‘0’ in the Log Page Attributes field, then the data beyond the last block
in Telemetry Controller-Initiated Data Area 3 Last Block is undefined.
• If the DA4S bit is set to ‘1’ in the Log Page Attributes field, then the data beyond the last block in
Telemetry Controller-Initiated Data Area 4 Last Block is undefined.
• If the DA4S bit is set to ‘1’ in the Log Page Attributes field and the Extended Telemetry Data Area
4 Supported (ETDAS) field is cleared to 0h in the Host Behavior Support feature, then the data
beyond the last block in Telemetry Controller-Initiated Data Area 3 Last Block is undefined.
If the host requests a data transfer that is not a multiple of 512 bytes, then the controller shall abort the
command with the status code of Invalid Field in Command.
Figure 216: Telemetry Controller-Initiated Log Page
Bytes Description
00 Log Page Identifier (LID): This field shall be set to 08h.
04:01 Reserved
07:05
IEEE OUI Identifier (IEEE):  Contains the Organization Unique Identifier (OUI) for the
controller vendor that is able to interpret the data. If this field is cleared to 0h, then no IEEE
OUI Identifier is present.  The OUI shall be a valid IEEE/RAC assigned identifier that is
registered at http://standards.ieee.org/develop/regauth/oui/public.html.
09:08
Telemetry Controller-Initiated Data Area 1 Last Block (TCDA1LB): Contains the value
of the last block of Telemetry Controller -Initiated Data Area 1. If the Telemetry Controller -
Initiated Data Area 1 does not contain data, then this field shall be cleared to 0h.
If this field is not 0h, then Telemetry Controller-Initiated Data Area 1 begins at block 1 and
ends at the block indicated in this field.
11:10
Telemetry Controller-Initiated Data Area 2 Last Block (TCDA2LB): Contains the value
of the last block of Telemetry Controller -Initiated Data Area 2. This value shall be greater
than or equal to the value in the Telemetry Controller-Initiated Data Area 1 Last Block field.
If this field is not 0h, then Telemetry Controller-Initiated Data Area 2 begins at block 1h and
ends at the block indicated in this field.
13:12
Telemetry Controller-Initiated Data Area 3 Last Block (TCDA3LB): Contains the value
of the last block of Telemetry Controller -Initiated Data Area 3. This value shall be greater
than or equal to the value in the Telemetry Controller-Initiated Data Area 2 Last Block field.
If this field is not 0h, then Telemetry Controller-Initiated Data Area 3 begins at block 1h and
ends at the block indicated in this field.
15:14 Reserved
19:16
Telemetry Controller-Initiated Data Area 4 Last Block (TCDA4LB): Contains the value
of the last block of Telemetry Controller-Initiated Data Area 4. If the DA4S bit is set to ‘1’ in
the Log Page Attributes field, then this value shall be greater than or equal to the value in
the Telemetry Controller-Initiated Data Area 3 Last Block field.
If this field is not 0h, then Telemetry Controller-Initiated Data Area 4 begins at block 1h and
ends at the block contained in this field.
380:20 Reserved
381
Telemetry Controller -Initiated Scope  (TCS): This field shall indicate the scope of the
Telemetry Controller -Initiated log  page. Implementations compliant with versions of this
specification later than 2.0 shall not set this field to a value of 0h.

Value Scope
00h Not reported
01h Controller
02h NVM subsystem
03h to FFh Reserved
