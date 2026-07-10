---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 275
headings: ["5.1.12.1.14.2.16 TCG Defined Event (Event Type DFh)", "5.1.12.1.15 Endurance Group Event Aggregate (Log Page Identifier 0Fh)"]
---

# Source page 275

NVM Express® Base Specification, Revision 2.1

253
Figure 251: Vendor Specific Event Data Type Codes
Value Description
01h
Event Name: The Vendor Specific Event Data field contains a null terminated ASCII string with a
vendor specific name for the value in the Vendor Specific Event Code field.
The value reported in this field shall be the same for every vendor specific event containing a vendor
specific event code that is the same as the value in the Vendor Specific Event Code field in this
event.
If a Vendor Specific Event Descriptor specifying this data type is reported, then that descriptor shall
be the first Vendor Specific Event Descriptor in that event.
02h ASCII String: The Vendor Specific Event Data field contains a null terminated ASCII string.
03h Binary: The Vendor Specific Event Data field contains binary data. The byte ordering in the binary
data is determined by the NVM subsystem vendor.
04h Signed Integer:  The Vendor Specific Event Data field contains a 64 -bit signed integer in two’s
complement form.
05h to FFh Reserved
5.1.12.1.14.2.16 TCG Defined Event (Event Type DFh)
The TCG Defined Event shall set the Persistent Event Log Event Format Header:
• Event Type field to DFh.
The Event Type Revision Field and the TCG Defined Event data are reserved for TCG.
5.1.12.1.15 Endurance Group Event Aggregate (Log Page Identifier 0Fh)
This log page indicates if an Endurance Group Event (refer to section  3.2.3) has occurred for a particular
Endurance Group. If an Endurance Group Event has occurred, the details of the particular event are
included in the Endurance Group Information log page for that Endurance Group. An asynchronous event
is generated when an entry for an Endurance Group is newly added to this log page.
If there is an enabled Endurance Group Event pending for an Endurance Group, then the Endurance Group
Event Aggregate log page includes an entry for that Endurance Group. The log page is an ordered list by
Endurance Group Identifier. For example, if Endurance Group Events are pending for Endurance Group 2,
1, and 7, then the log page shall have entries in numerical order of 1, 2, and 7. A particular Endurance
Group entry is removed from this log page after the Get Log Page command is completed successfully with
the Retain Asynchronous Event bit cleared to ‘0’ for the Endurance Group Information log page for that
Endurance Group.
The log page size is limited by the Endurance Group Identifier Maximum value reported in the Identify
Controller data structure (refer to Figure 312). If the host reads beyond the end of the log page, zeroes are
returned. The log page is defined in Figure 252.
Figure 252: Endurance Group Event Aggregate Log Page
Bytes Description
Header
07:00
Number of Entries (NUMENT): This field indicates the number of entries in the list. The maximum
number of entries in the list corresponds to the Endurance Group Identifier Maximum field reported
in the Identify Controller data structure. A value of 0h indicates there are no entries in the list.
Endurance Group Identified List
09:08 Entry 1: Indicates the Endurance Group that has an Endurance Group Event pending that has
the numerically smallest Endurance Group Identifier, if any.
11:10 Entry 2: Indicates the Endurance Group that has an Endurance Group Event pending that has
the second numerically smallest Endurance Group Identifier, if any.
… …
2*NUMENT+7:
2*NUMENT+6
Entry NUMENT: Indicates the Endurance Group that has an Endurance Group Event pending
that has the numerically largest Endurance Group Identifier, if any.
