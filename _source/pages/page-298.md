---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 298
headings: ["5.1.12.1.31 Flexible Data Placement (FDP) Events (Log Page Identifier 23h)"]
---

# Source page 298

NVM Express® Base Specification, Revision 2.1

276
Figure 286: FDP Statistics Log Page
Bytes Description
15:00
Host Bytes with Metadata Written (HBMW): Contains the total number of bytes of user data the host
has written to the specified Endurance Group as part of processing I/O commands as defined in the
appropriate I/O Command Set specification. This value does not include controller writes due to internal
operations such as garbage collection.
This field shall not wrap once the value FFFFFFFF_FFFFFFFF_FFFFFFFF_FFFFFFFFh is reached.
If the Flexible Data Placement Feature value is modified by a Set Features command, then this field
shall be cleared to 0h.
31:16
Media Bytes with Metadata Written (MBMW):  Contains the total number of bytes of user data that
have been written to the specified Endurance Group including both host and controller writes (e.g.,
garbage collection and background media scan operations) as part of processing as defined in the
appropriate I/O Command Set specification.
This field shall not wrap once the value FFFFFFFF_FFFFFFFF_FFFFFFFF_FFFFFFFFh is reached.
If the Flexible Data Placement Feature value is modified by a Set Features command, then this field
shall be cleared to 0h.
47:32
Media Bytes Erased (MBE):  Contains the total number of bytes of data that have been erased in the
specified Endurance Group.
This field shall not wrap once the value FFFFFFFF_FFFFFFFF_FFFFFFFF_FFFFFFFFh is reached.
If the Flexible Data Placement Feature value is modified by a Set Features command, then this field
shall be cleared to 0h.
63:48 Reserved
The values reported in this log page are not cleared to 0h by a firmware update.
The values reported in this log page include operations on all namespaces that existed in the specified
Endurance Group since the Flexible Data Placement was last enabled in that Endurance Group (refer to
section 5.1.25.1.20).
5.1.12.1.31 Flexible Data Placement (FDP) Events (Log Page Identifier 23h)
The FDP Events log page is used to provide information about events affecting Reclaim Units and media
usage in an Endurance Group that has Flexible Data Placement enabled. The log page is 4 KiB bytes in
size. The Endurance Group is specified by the Endurance Group Identifier in the Log Specific Identifier field
as defined in Figure 217.
FDP events are associated with Reclaim Unit Handles. An FDP event shall only be reported if the FDP
event occurs and that FDP event is enabled for that Reclaim Unit Handle (refer to section 5.1.25.1.21).
If:
• an FDP event occurs;
• that event is enabled; and
• the number of events recorded is the maximum supported,
then the oldest event shall be discarded.
The FDP Event Type bit (refer to Figure 287) specifies whether the controller reports host events or
controller events in the log page. The controller shall not report both host events and controller events in
the log page.
If Flexible Data Placement is enabled in the specified Endurance Group and a Get Log Page command
specifies this log page, then the NSID field in that command is reserved.
If Flexible Data Placement is disabled in the specified Endurance Group, then a Get Log Page command
that specifies this log page shall be aborted with a status code of FDP Disabled.
