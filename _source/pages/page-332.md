---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 332
headings: []
---

# Source page 332

NVM Express® Base Specification, Revision 2.1

310
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
261 M M M
Log Page Attributes (LPA): This field indicates optional attributes for log pages that are
accessed via the Get Log Page command.
Bits Description
7 Reserved
6
Data Area 4 Support (DA4S): If this bit is  set to ‘1’, then the controller
supports Data Area 4 for the Telemetry Host -Initiated and Telemetry
Controller-Initiated log. If this bit is cleared to ’0’, then the controller does not
support Data Area 4 for the Telemetry Host -Initiated and Telemetry
Controller-Initiated log pages.
5
Miscellaneous Log Page Support (MLPS): If this bit is set to ‘1’, then the
controller supports:
• the Supported Log Pages log page (Log Page Identifier 0h);
• returning the scope of each command in the Commands Supported
and Effects log page (Log Page Identifier 05h);
• the Feature Identifiers Supported and Effects log page ( Log Page
Identifier 12h); and
• the NVMe -MI Commands Supported and Effects log page ( Log
Page Identifier 13h).
If this bit is cleared to ‘0’, then the controller:
• does not support returning the scope of each command in the
Commands Supported and Effects log page;
• may support the Supported Log Pages log page;
• may support the Feature Identifiers Supported and Effects log
page; and
• may support the NVMe -MI Commands Supported and Effects log
page.
4
Persistent Event Support (PES): If this bit is set to ‘1’, then the controller
supports the Persistent Event log. If this bit is cleared to ‘0’, then the
controller does not support the Persistent Event log page.
3
Telemetry Support (TS): If this bit is set to ‘1’, then the controller supports
the Telemetry Host-Initiated and Telemetry Controller-Initiated log pages and
sending Telemetry Log Notices. If this bit is cleared to ’0’, then the controller
does not support the Telemetry Host -Initiated and Telemetry Controller -
Initiated log pages and Telemetry Log Notice events.
2
Log Page Extended Data Support (LPEDS): If this bit is set to ‘1’, then the
controller supports extended data for the Get Log Page command (including
extended Number of Dwords and Log Page Offset fields). If this bit is cleared
to ‘0’, then the controller does not support extended data for the Get Log
Page command.
1
Commands Supported and Effects Support (CSES): If this bit is set to ‘1’,
then the controller supports the Commands Supported and Effects log page.
If this bit is cleared to ‘0’, then the controller does not support the Commands
Supported and Effects log page.
0
SMART Support (SMARTS): If this bit is set to ‘1’, then the controller
supports the SMART / Health Information log page on a per namespace
basis. If this bit is cleared to ‘0’, then the controller does not support the
SMART / Health Information log page on a per namespace basis.

262 M M M Error Log Page Entries (ELPE): This field indicates the maximum number of Error
Information Log Entries that are stored by the controller. This field is a 0’s based value.
