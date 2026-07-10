---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 242
headings: []
---

# Source page 242

NVM Express® Base Specification, Revision 2.1

220
Figure 213: Telemetry Host-Initiated Log Specific Parameter Field
Bits Description
08
Create Telemetry Host-Initiated Data (CTHID): If this bit is set to ‘1’, then the controller shall capture
the Telemetry Host -Initiated Data representing the internal state of the controller at the time the
associated Get Log Page command is processed. If this bit is cleared to ‘0’, then the controller shall not
update the Telemetry Host -Initiated Data. The Host -Initiated Data shall not change until the controller
processes:
a) a subsequent Telemetry Host-Initiated Log with this bit set to ‘1’;
b) a Firmware Commit command; or
c) a power on reset.
The Telemetry Host-Initiated Data consists of:
a) Three areas, if the Data Area 4 Support (DA4S) bit is cleared to ‘0’ in the Log Page Attributes field:
Telemetry Host-Initiated Data Area 1, Telemetry Host -Initiated Data Area 2, and Telemetry Host -
Initiated Data Area 3; or
b) Four areas, if the DA4S bit is set to ‘1’ in  the Log Page Attributes field: Telemetry Host -Initiated
Data Area 1, Telemetry Host -Initiated Data Area 2, Telemetry Host -Initiated Data Area 3 and
Telemetry Host-Initiated Data Area 4.
All areas start at Telemetry Host -Initiated Data Area Block 1. The last block of each area is indicated in
Telemetry Host-Initiated Data Area y Last Block, respectively. The telemetry data captured and the size of
that data is implementation dependent.
The size of the log page is variable and:
• If the DA4S bit is cleared to ‘0’ in the Log Page Attributes field, the size may be calculated using
the Telemetry Host-Initiated Data Area 3 Last Block field.
• If the DA4S bit is set to ‘1’ in the Log Page Attributes field and the Extended Telemetry Data Area
4 Supported (ETDAS) field is set to 1h in the Host Behavior Support feature (refer to section
5.1.25.1.14), then the size of the log page may be calculated using the Telemetry Host -Initiated
Data Area 4 Last Block field.
• If the DA4S bit is set to ‘1’ in the Log Page Attributes field and the Extended Telemetry Data Area
4 Supported (ETDAS) field is cleared to 0h in the Host Behavior Support feature (refer to section
5.1.25.1.14), then the size of the log page may be calculated using the Telemetry Host -Initiated
Data Area 3 Last Block field.
The controller shall return data for all blocks requested:
• If the DA4S bit is cleared to ‘0’ in the Log Page Attributes field, then the data beyond the last block
in Telemetry Host-Initiated Data Area 3 Last Block is undefined.
• If the DA4S bit is set to ‘1’ in the Log Page Attributes field and the Extended Telemetry Data Area
4 Supported (ETDAS) field is set to 1h in the Host Behavior Support feature, then the data beyond
the last block in Telemetry Host-Initiated Data Area 4 Last Block is undefined.
• If the DA4S bit is set to ‘1’ in the Log Page Attributes field and the Extended Telemetry Data Area
4 Supported (ETDAS) field is cleared to 0h in the Host Behavior Support feature, then the data
beyond the last block in Telemetry Host-Initiated Data Area 3 Last Block is undefined.
If the host requests a data transfer that is not a multiple of 512 bytes, then the controller shall return an
error of Invalid Field in Command.
If the MCDAS bit (refer to Figure 215) is set to ‘1’ and a Get Log Page command with the Create Telemetry
Host-Initiated Data bit is set to ‘1’, then the maximum data area to be created in the Telemetry Host-Initiated
log page shall be less than or equal to the MCDA field in the Log Specific Parameter field in Command
Dword 10 (refer to Figure 213).
