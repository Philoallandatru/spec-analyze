---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 605
headings: []
---

# Source page 605

NVM Express® Base Specification, Revision 2.1

583
b) Four Telemetry Data Areas (i.e., small, medium, large and extra -large) If the DA4S bit is set to ‘1’
and the Extended Telemetry Data Area 4 Supported (ETDAS) field is set to 1h in the Host Behavior
Support feature (refer to section 5.1.25.1.14).
All telemetry data areas start at Telemetry Data Block 1.
Each Telemetry Data Area shall represent the controller’s internal state at the time the telemetry data was
captured.
Each Telemetry Data Area is intended to capture a richer set of data to aid in resolution of issues. Telemetry
Data Area 1 is intended to have a small size payload (i.e., the first phase), Telemetry Data Area 2 is intended
to have a medium size payload (i.e., the second phase), and Telemetry Data Area 3 is intended to have a
large size payload ( i.e., the third phase). Telemetry Data Area 4 is intended to have an extra -large size
payload (i.e., the fourth phase). The size of each Telemetry Data Area is vendor specific and may change
on each data collection. When possible, the host should retrieve the payload for all supported Telemetry
Data Areas to enable the best diagnosis of the issue(s).
The preparation, collection, and submission of telemetry data is similar for host -initiated and controller -
initiated data; the primary difference is the trigger for the collection. The operational model for telemetry is:
1. The host identifies controller support for Telemetry log pages in the Identify Controller data
structure;
2. The host may indicate the support for the Telemetry Host -Initiated Data Area 4 and Telemetry
Controller-Initiated Data Area 4 by setting the Extended Telemetry Data Area 4 Supported (ETDAS)
field to 1h in the Host Behavior Support feature (refer to section 5.1.25.1.14);
3. The host prepares an area to store telemetry data if needed;
4. To receive notification that controller -initiated telemetry data is available, the host enables
Telemetry Log Notices using the Asynchronous Event Configuration feature  (refer to section
5.1.25.1.5); and
5. If the host decides to collect host -initiated telemetry data or the controller signals that controller -
initiated telemetry data is available:
a. The host reads the appropriate blocks of the Telemetry Data Area from the Telemetry Host-
Initiated log page (refer to section 5.1.12.1.8) or the Telemetry Controller-Initiated log page
(refer to section 5.1.12.1.9). If possible, the host should collect Telemetry Data Area 1, 2, 3 ,
and 4. The host reads the log in 512 byte Telemetry Data Block units (i.e., a starting offset that
is a multiple of 512, and a length that is a multiple of 512) . The host should set the Retain
Asynchronous Event bit to ‘1’;
b. The host re -reads the header of the log page and ensures that the Telemetry Host -Initiated
Data Generation Number field from the  Telemetry Host-Initiated log page or the Telemetry
Controller-Initiated Data Generation Number field in the Telemetry Controller-Initiated log page
matches the original value read. If these values do not match, then the data captured is not
consistent and should be re-read from the log page with the Retain Asynchronous Event bit set
to ‘1’;
c. If the host is reading the controller -initiated log, then the host ensures that the Telemetry
Controller-Initiated Data Available field is still set to 1h after reading the appropriate blocks of
the Telemetry Data Area because the Telemetry Controller -Initiated Data Available field may
have been cleared to 0h by another entity while the log page was being read;
d. If the host is reading the Telemetry Controller-Initiated log page, then the host reads any portion
of that log page with the Retain Asynchronous Event bit cleared to ‘0’ to indicate to the controller
that the host has completed reading the Telemetry Controller-Initiated log page; and
e. When all telemetry data has been saved, the data should be forwarded to the manufacturer of
the controller.
The trigger for the collection for host -initiated data is typically a system crash, but may also be initiated
during normal operation. The host proceeds with a host -initiated data collection by submitting the Get Log
Page command for the Telemetry Host-Initiated log page with the Create Telemetry Host-Initiated Data bit
