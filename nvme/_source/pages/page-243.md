---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 243
headings: []
---

# Source page 243

NVM Express® Base Specification, Revision 2.1

221
Figure 214: Telemetry Host-Initiated Log Page
Bytes Description
00 Log Page Identifier (LID): This field shall be set to 07h.
04:01 Reserved
07:05
IEEE OUI Identifier (IEEE):  Contains the Organization Unique Identifier (OUI) for the
controller vendor that is able to interpret the data. If this field is cleared to 0h, then no IEEE
OUI Identifier is present.  The OUI shall be a valid IEEE/RAC assigned identifier that is
registered at http://standards.ieee.org/develop/regauth/oui/public.html.
09:08
Telemetry Host-Initiated Data Area 1 Last Block  (THDA1LB): Contains the value of the
last block of Telemetry Host-Initiated Data Area 1. If the Telemetry Host -Initiated Data Area
1 does not contain data, then this field shall be cleared to 0h.
If this field is not 0h, then Telemetry Host-Initiated Data Area 1 begins at block 1h and ends
at the block indicated in this field.
11:10
Telemetry Host-Initiated Data Area 2 Last Block  (THDA2LB): Contains the value of the
last block of Telemetry Host-Initiated Data Area 2. This value shall be greater than or equal
to the value in the Telemetry Host-Initiated Data Area 1 Last Block field.
If this field is not 0h, then Telemetry Host-Initiated Data Area 2 begins at block 1h and ends
at the block indicated in this field.
13:12
Telemetry Host-Initiated Data Area 3 Last Block  (THDA3LB): Contains the value of the
last block of Telemetry Host-Initiated Data Area 3. This value shall be greater than or equal
to the value in the Telemetry Host-Initiated Data Area 2 Last Block field.
If this field is not 0h, then Telemetry Host-Initiated Data Area 3 begins at block 1h and ends
at the block contained in this field.
15:14 Reserved
19:16
Telemetry Host-Initiated Data Area 4 Last Block  (THDA4LB): Contains the value of the
last block of Telemetry Host -Initiated Data Area 4. If the Data Area 4 Support (DA4S) bit is
set to ‘1’ in the Log Page Attributes field is set to ‘1’, then this value shall be greater than or
equal to the value in the Telemetry Host-Initiated Data Area 3 Last Block field.
If this field is not 0h, then Telemetry Host-Initiated Data Area 4 begins at block 1h and ends
at the block contained in this field.
379:20 Reserved
380
Telemetry Host-Initiated Scope (THS): This field shall indicate the scope of the Telemetry
Host-Initiated log page. Implementations compliant with versions of this specification later
than 2.0 shall not set this field to a value of 0h.

Value Scope
00h Not reported
01h Controller
02h NVM subsystem
03h to FFh Reserved
381
Telemetry Host -Initiated Data Generation Number  (THDGN): Contains a value that is
incremented each time the internal controller state or internal NVM subsystem state for this
log page is captured. If the value of this field is FFh, then the field shall be cleared to 0h when
incremented (i.e., rolls over to 0h).
382
Telemetry Controller-Initiated Data Available  (TCDA): Contains the value of Telemetry
Controller-Initiated Data Available field in the Telemetry Controller -Initiated log (refer to
Figure 216).
383
Telemetry Controller-Initiated Data Generation Number (TCDGN): Contains the value of
the Telemetry Controller-Initiated Data Generation Number field in the Telemetry Controller-
Initiated log (refer to Figure 216).
511:384
Reason Identifier (RID): Contains a vendor specific identifier that describes the operating
conditions of the controller at the time of capture. The Reason Identifier field should provide
an identification of unique operating conditions of the controller.
1023:512 Telemetry Host-Initiated Data Block 1 (THDB1): Contains Telemetry Data Block 1 for the
Telemetry Host-Initiated Log.
1535:1024 Telemetry Host-Initiated Data Block 2 (THDB2): Contains Telemetry Data Block 2 for the
Telemetry Host-Initiated Log.
