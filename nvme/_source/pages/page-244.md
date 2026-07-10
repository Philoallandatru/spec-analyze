---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 244
headings: ["5.1.12.1.8.1 Telemetry Host-Initiated LID Specific Parameter Field", "5.1.12.1.9 Telemetry Controller-Initiated (Log Page Identifier 08h)"]
---

# Source page 244

NVM Express® Base Specification, Revision 2.1

222
Figure 214: Telemetry Host-Initiated Log Page
Bytes Description
… …
(n*512)+511:(n*512) Telemetry Host-Initiated Data Block n (THDBN): Contains Telemetry Data Block n for the
Telemetry Host-Initiated Log.
 Telemetry Host-Initiated LID Specific Parameter Field
Figure 215 specifies the format for the LID Specific Parameter field in the Supported Log Pages log page
(refer to section 5.1.12.1.1) for the Telemetry Host-Initiated log page.
Figure 215: Telemetry Host-Initiated Log Page - LID Specific Parameter Field
Bits Description
15:1 Reserved
0
Maximum Created Data Area Support (MCDAS): If this bit is set to ‘1’, then the controller supports the
Maximum Created Data Area field in Log Specific Parameter field for a Telemetry Host-Initiated log page. If
this bit is cleared to ‘0’, then the controller does not support the Maximum Created Data Area field in Log
Specific Parameter field for a Telemetry Host-Initiated log page.
5.1.12.1.9 Telemetry Controller-Initiated (Log Page Identifier 08h)
This log page consists of a header  (i.e., bytes 511:0 of the log page)  describing the log and zero or more
Telemetry Data Blocks (refer to section 8.1.27). The header shall always be available even if there is no
Telemetry Controller-Initiated Data available. All Telemetry Data Blocks are 512 bytes in size. This log is a
controller-initiated capture of the controller’s internal state  or NVM subsystem’s internal state . The
Telemetry Controller-Initiated Data for Data Area 1 through Data Area 3 shall persist across all resets. The
Telemetry Controller-Initiated Data for Data Area 4 may persist across Controller Level Resets. If the host
specifies a Log Page Offset Lower value that is not a multiple of 512 bytes in the Get Log Page command
for this log, then the controller shall return an error of Invalid Field in Command.
The Telemetry Controller-Initiated Data consists of:
a) three areas, if the Data Area 4 Support (DA4S) bit is cleared to ‘0’ in the Log Page Attributes field:
Telemetry Controller -Initiated Data Area 1, Telemetry Controller -Initiated Data Area 2, and
Telemetry Controller-Initiated Data Area 3; or
b) four areas, if the DA4S bit is set to ‘1’ in the Log Page Attributes field: Telemetry Controller-Initiated
Data Area 1, Telemetry Controller-Initiated Data Area 2, Telemetry Controller-Initiated Data Area 3
and Telemetry Controller-Initiated Data Area 4.
All areas start at Telemetry Controller-Initiated Data Area Block 1. The last block of each area is indicated
in the Telemetry Controller-Initiated Data Area y Last Block, respectively. The telemetry data captured and
its size is implementation dependent.
The size of the log page is variable and:
• If the DA4S bit is cleared to ‘0’ in the Log Page Attributes field, the size may be calculated using
the Telemetry Controller-Initiated Data Area 3 Last Block field.
• If the DA4S bit is set to ‘1’ in the Log Page Attributes field and the Extended Telemetry Data Area
4 Supported (ETDAS) field is set to 1h in the Host Behavior Support feature (refer to section
5.1.25.1.14), then the size of the log page may be calculated using the Telemetry Controller -
Initiated Data Area 4 Last Block field.
• If the DA4S bit is set to ‘1’ in the Log Page Attributes field and the Extended Telemetry Data Area
4 Supported (ETDAS) field is cleared to 0h in the Host Behavior Support feature (refer to section
5.1.25.1.14), then the size of the log page may be calculated using the Telemetry Controller -
Initiated Data Area 3 Last Block field.
The controller shall return data for all blocks requested:
