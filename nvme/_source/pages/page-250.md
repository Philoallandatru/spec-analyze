---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 250
headings: ["5.1.12.1.12 Predictable Latency Event Aggregate (Log Page Identifier 0Bh)", "5.1.12.1.13 Asymmetric Namespace Access (Log Page Identifier 0Ch)"]
---

# Source page 250

NVM Express® Base Specification, Revision 2.1

228
Figure 220: Predictable Latency Per NVM Set Log Page
Bytes Description
143:136
DTWIN Writes Estimate (DTWWE): Indicates a reliable estimate of the number of writes in units
of the Optimal Write Size remaining in the current Deterministic Window, if applicable. This value
decrements from DTWIN Writes Typical to 0h based on host write activity and operating
conditions. Refer to section 8.1.18.1.
151:144 DTWIN Time Estimate  (DTWTE): Indicates a reliable estimate of the time in milliseconds
remaining in the current Deterministic Window, if applicable. Refer to section 8.1.18.1.
511:152 Reserved
5.1.12.1.12 Predictable Latency Event Aggregate (Log Page Identifier 0Bh)
This log page indicates if a Predictable Latency Event (refer to section 8.1.18) has occurred for a particular
NVM Set. If a Predictable Latency Event has occurred, the details of the particular event are included in the
Predictable Latency Per NVM Set log page for that NVM Set. An asynchronous event is generated when
an entry for an NVM Set is newly added to this log page.
This log page shall not contain an entry (i.e., an NVM Set Identifier) that is cleared to 0h.
If there is an enabled Predictable Latency Event pending for an NVM Set, then the Predictable Latency
Event Aggregate log page includes an entry for that NVM Set. The log page is an ordered list by NVM Set
Identifier. For example, if Predictable Latency Events are pending for NVM Set 27, 13, and 17, then the log
page shall have entries in numerical order of 13, 17, and 27. A particular NVM Set is removed from this log
page after the Get Log Page  command with the Retain Asynchronous Event bit cleared to ‘0’ for the
Predictable Latency Per NVM Set log page for that NVM Set is completed successfully.
The log page size is limited by the NVM Set Identifier Maximum value reported in the Identify Controller
data structure (refer to Figure 312). If the host reads beyond the end of the log page, zero es are returned.
The log page is defined in Figure 221.
Figure 221: Predictable Latency Event Aggregate Log Page
Bytes Description
Header
07:00
Number of Entries (NUMENT): This field indicates the number of entries in the list. The
maximum number of entries in the list corresponds to the NVM Set Identifier Maximum  field
reported in the Identify Controller data structure. A value of 0 h indicates there are no entries
in the list.
NVM Set Identifier List
09:08 Entry 1: Indicates the NVM Set that has a Predictable Latency Event pending that has the
numerically smallest NVM Set Identifier, if any.
11:10 Entry 2: Indicates the NVM Set that has a Predictable Latency Event pending that has the
second numerically smallest NVM Set Identifier, if any.
… …
(NUMENT*2) + 9:
(NEMENT*2) + 8
Entry NUMENT: Indicates the NVM Set that has a Predictable Latency Event pending that
has the numerically largest NVM Set Identifier, if any.
5.1.12.1.13 Asymmetric Namespace Access (Log Page Identifier 0Ch)
This log page consists of a header describing the log and descriptors containing the asymmetric namespace
access information for ANA Groups (refer to section 8.1.1.2) that contain namespaces that are attached to
the controller processing the command. If ANA Reporting (refer to section 8.1.1) is supported, this log page
is supported. ANA Group Descriptors shall be returned in ascending ANA Group Identifier order.
If the Index Offset Supported bit is cleared to ‘0’ in the LID Support and Effects data structure for this log
page (refer to Figure 204), then:
