---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 256
headings: []
---

# Source page 256

NVM Express® Base Specification, Revision 2.1

234
Figure 226: Persistent Event Log Page
Bytes Description
Persistent Event Log Header
00 Log Page Identifier (LID): This field shall be set to 0Dh.
03:01 Reserved
07:04 Total Number of Events (TNEV): Contains the number of event entries in this log page.
15:08 Total Log Length (TLL):  Contains the total number of bytes of persistent event log page data
available, including the header.
16
Log Revision  (LREV): Contains a number indicating the revision of the Get Log Page data
structure that this log page data complies with. Shall be set to 03h. This revision applies to the
Persistent Event Log and the Persistent Event Format (refer to Figure 227). This revision does
not apply to the contents of the Event Data field in the Persistent Event Format as that field is
covered by the Event Type Revision (refer to Figure 227).
17 Reserved
19:18 Log Header Length (LHL): This field contains the length in bytes of the log header information
that follows. The total length of the log header in bytes is the value in this field plus 20.
27:20
Timestamp (TSTMP): Shall contain a timestamp of the time at which the persistent event log
reporting context was established . The value returned shall use  the Timestamp data structure
defined in Figure 396.
43:28
Power on Hours (POH):  This field indicates the number of power -on hours at the time the
Persistent Event log was retrieved. This may not include time that the controller was powered
and in a non-operational state.
51:44 Power Cycle Count (PWRCC): Contains the number of power cycles for this controller.
53:52 PCI Vendor ID (VID): This is the same value as reported in the Identify Controller data structure
PCI Vendor ID field (i.e., bytes 01:00).
55:54 PCI Subsystem Vendor ID (SSVID): This is the same value as reported in the Identify Controller
data structure PCI Subsystem Vendor ID field (i.e., bytes 03:02).
75:56 Serial Number (SN): This field contains the same value as reported in the Serial Number field
of the Identify Controller data structure, bytes 23:04.
115:76 Model Number (MN): This field contains the same value as reported in the Model Number field
of the Identify Controller data structure, bytes 63:24.
371:116
NVM Subsystem NVMe Qualified Name (SUBNQN):  This field contains the same value as
reported in the NVM Subsystem NVMe Qualified Name field of the Identify Controller data
structure, bytes 1023:768. If the NVM Subsystem NVMe Qualified Name field of the Identify
Controller data structure is not supported, then all bytes of this field shall be cleared to 0h.
373:372
Generation Number (GNUM): Contains a value that is incremented each time a persistent event
log reporting context is established and the log page returns different data than when this log
page last established a reporting context. If the value of this field is FFFFh, then the field  shall
be cleared to 0h when incremented (i.e., rolls over to 0h).
