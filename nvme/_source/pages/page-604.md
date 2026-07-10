---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 604
headings: ["8.1.26 Standard Vendor Specific Command Format", "8.1.27 Telemetry"]
---

# Source page 604

NVM Express® Base Specification, Revision 2.1

582
The host enables the SQ Associations capability by creating an association between an NVM Set and a
Submission Queue at the time the Submission Queue is created (e.g., with a Create I/O Submission Queue
command (refer to section 5.2.2) or a Connect command (refer to section 6.3)).
For the SQ Associations capability to yield benefits, the host is required to:
a) create an association between each Submission Queue and some NVM Set; and
b) only issue I/O commands to Submission Queues that have an association with the NVM Set that
contains the namespace associated with the Namespace Identifier specified in that I/O command.
While this capability is enabled, failure to follow the specified operating rules may impact Predictable
Latency (refer to section 8.1.18).
 Standard Vendor Specific Command Format
Controllers may support the standard Vendor Specific command format defined in Figure 82. Host storage
drivers may use the Number of Dwords fields to ensure that the application is not corrupting physical
memory (e.g. , overflowing a data buffer).  The controller indicates support of this format in the Identify
Controller data structure in Figure 312; refer to the Admin Vendor Specific Command Configuration  field
and the I/O Command Set Vendor Specific Command Configuration field.
 Telemetry
Telemetry enables manufacturers to collect internal data logs to improve the functionality and reliability of
products. The telemetry data collection may be initiated by the host or by the controller. The data is returned
in the Telemetry Host -Initiated lo g page or the Telemetry Controller -Initiated log page (refer to section
5.1.12.1.8 and 5.1.12.1.9). The data captured is vendor specific. The telemetry feature defines the
mechanism to collect the vendor specific data. The controller indicates support for the telemetry log pages
and for the Data Area 4 size in the Log Page Attributes (LPA) field in the Identify Controller data structure
(refer to Figure 312).
An important aspect to discovering issues by collecting telemetry data is the ability to qualify distinct issues
that are being collected. The ability to create a one to one mapping of issues to data collections is essential.
If a one to one mapping is not established, there is the risk that several payload collections appear distinct
but are actually all caused by the same issue.  Conversely, a single payload collection may have payloads
caused by several issues mixed together creating additional complexity in determining  the root cause. As
a result, flexibility in size is provided in the collection of telemetry payloads and a three phase process is
typically used.
The first phase establishes that an issue exists and is best accomplished by collecting a minimum set of
data to identify the issue as being distinct from other issues.  Once the number of instances of an issue
establish an investigation, another phase may be necessary to collect actionable information. In the second
phase, a targeted collection of more in depth medium size payloads are gathered and analyzed to identify
the source of the problem.
If the small or medium sized telemetry data collection provides insufficient information, a third phase may
be employed to collect additional details. If the Data Area 4 Support (DA4S) bit  is cleared to ‘0’ in the Log
Page Attributes field, then the third phase provides the largest and most complete payload to diagnose the
issue. If the DA4S bit is set to ‘1’ and the Extended Telemetry Data Area 4 Supported (ETDAS) field is set
to 1h in the Host Behavior Support feature (refer to section 5.1.25.1.14) then a fourth phase may be
employed to collect the largest and most complete payload to diagnose the issue. If Data Area 4 is created,
then Data Area 3 of non-zero length shall also be created and populated as part of data collection.
There are two telemetry data logs (i.e., Telemetry Host-Initiated log page and Telemetry Controller-Initiated
log page) defined. Each telemetry data log is made up of a single set of Telemetry Data Blocks.  Each
Telemetry Data Block is 512 bytes in size. Telemetry data is returned (refer to section 5.1.12.1.8 and section
5.1.12.1.9) in units of Telemetry Data Blocks. Each telemetry data log is segmented into :
a) Three Telemetry Data Areas (i.e., small, medium, and large), if the DA4S bit is cleared to ‘0’; or
