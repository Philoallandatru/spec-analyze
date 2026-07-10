---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 607
headings: []
---

# Source page 607

NVM Express® Base Specification, Revision 2.1

585
Figure 656: Telemetry Log Example – All Data Areas Populated
Block Number Telemetry Host-Initiated Data Areas
1

65

Data Area 1 *

Data Area 2 *

Data Area 3 *




1,000

Data Area 2 +
continued

Data Area 3 +
continued






30,000


Data Area 3
continued
* Data Area 1, Data Area 2, and Data Area 3 contain the same telemetry data in blocks 1 through 65.
+ Data Area 2 and Data Area 3 contain the same telemetry data in blocks 66 through 1 ,000.
When only the second data area is populated, then the Telemetry Host -Initiated log page has no data in
Telemetry Data Area 1 shown by having its corresponding last block value cleared to 0h, and no additional
data in Telemetry Data Area 3 shown by having its corresponding last block value set to the same value as
the last block value for Telemetry Data Area 2. For example, the following values correspond to the layout
shown in Figure 657:
• Telemetry Host-Initiated Data Area 1 Last Block = 0;
• Telemetry Host-Initiated Data Area 2 Last Block = 1,000; and
• Telemetry Host-Initiated Data Area 3 Last Block = 1,000.
As a result of telemetry data areas being made up of a single set of Telemetry Data Blocks starting at
Telemetry Data Block 1, the telemetry data contained in Telemetry Data Block 1 through Telemetry Data
Block 1,000 of data area of data area 2 and data area 3 is the same.
