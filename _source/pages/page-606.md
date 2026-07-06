---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 606
headings: ["8.1.27.1 Telemetry Data Collection Examples (Informative)"]
---

# Source page 606

NVM Express® Base Specification, Revision 2.1

584
set to ‘1’ in the Log Specific Parameter field. The controller should complete the command quickly (e.g., in
less than one second) to avoid a user rebooting the system prior to completion of the data collection.
The NVM subsystem is allowed to provide a Telemetry Host-Initiated log page per controller or a shared
Telemetry Host-Initiated log page across all controllers in the NVM subsystem. If a shared Telemetry Host-
Initiated log page is implemented, the Telemetry Host -Initiated Data Generation Number field in the
Telemetry Host-Initiated log page is used to allow the host to detect that the Telemetry Host-Initiated log
page has been changed by:
• a host through a different controller; or
• a Management Controller through a Management Endpoint (refer to the NVM Express
Management Interface Specification).
The controller notifies the host to collect controller-initiated data through the completion of an Asynchronous
Event Request command with an Asynchronous Event Type of Notice that indicates a Telemetry Log
Changed event. The host may also determine contro ller-initiated data is available via the Telemetry
Controller-Initiated Data Available field in the Telemetry Host -Initiated or the Telemetry Controller-Initiated
log pages. The host proceeds with a controller -initiated data collection by submitting the Ge t Log Page
command for the Telemetry Controller-Initiated log page. Once the host has started reading the Telemetry
Controller-Initiated log page, the controller should avoid modifying the controller-initiated data until the host
has finished reading all c ontroller-initiated data. The amount of time for the host to read the controller -
initiated data is vendor specific.
Since there is only one set of controller-initiated data, the controller is responsible for prioritizing the version
of the controller -initiated data that is available for the host to collect. When the controller replaces the
controller-initiated data with new controller -initiated data , the controller shall increment the Telemetry
Controller-Initiated Data Generation Number field. The host needs to ensure that the Telemetry Controller-
Initiated Data Generation Number field has not changed between the start and completion of the controller-
initiated data collection to ensure the data captured is consistent.
 Telemetry Data Collection Examples (Informative)
This section includes several examples of Telemetry Host -Initiated Data Areas for illustration. The same
concepts apply to the Telemetry Controller-Initiated Data Areas.
If a Telemetry Host-Initiated log page has no data for collection , then the following fields are all cleared to
0h:
• Telemetry Host-Initiated Data Area 1 Last Block = 0;
• Telemetry Host-Initiated Data Area 2 Last Block = 0; and
• Telemetry Host-Initiated Data Area 3 Last Block = 0.
When all three telemetry data areas are populated, then the Telemetry Host-Initiated log page has different
values in each of the Telemetry Host -Initiated Data Area n Last Block fields.  For example, the following
values correspond to the layout shown in Figure 656:
• Telemetry Host-Initiated Data Area 1 Last Block = 65;
• Telemetry Host-Initiated Data Area 2 Last Block = 1,000; and
• Telemetry Host-Initiated Data Area 3 Last Block = 30,000.
As a result of telemetry data areas being made up of a single set of Telemetry Data Blocks starting at
Telemetry Data Block 1, the telemetry data contained in Telemetry Data Block 1 through Telemetry Data
Block 65 of data area 1, data area 2, and data area 3 is the same. In addition, the telemetry data contained
in Telemetry Data Block 66 through Telemetry Data Block 1,000 of data area 2 and data area 3 is the same.
