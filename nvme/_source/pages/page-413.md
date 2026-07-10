---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 413
headings: ["5.1.25.1.21 Flexible Data Placement Events (Feature Identifier 1Eh)"]
---

# Source page 413

NVM Express® Base Specification, Revision 2.1

391
5.1.25.1.21 Flexible Data Placement Events (Feature Identifier 1Eh)
This Feature controls if a controller generates Flexible Data Placement (FDP) events associated with a
specific Reclaim Unit Handle. The Reclaim Unit Handle is the Reclaim Unit Handle associated with the
specified Placement Handle for the specified namespace. For a Set Features command, the data buffer
contains the list of FDP Event Types that are to be enabled or disabled as defined in Figure 420.
The attribute is specified in Command Dword 12. Effects of enabling and disabling FDP events on the
Reclaim Unit Handles are described in section 8.1.10.
If Flexible Data Placement is disabled, then the controller shall abort any Set Feature command or Get
Feature command specifying this Feature with a status code of FDP Disabled.
If the NSID field is set to FFFFFFFFh, then the controller shall abort the command with a status code of
Invalid Field in Command.
If a Get Features command specifying this Feature is successfully completed, then the attributes for the
supported list of FDP Event Types described in Figure 421 is returned in the data buffer and Dword 0 of
the completion queue entry contains the value of NOET (i.e.,  the number of supported FDP Event Types
that are contained in the data buffer) . Completion queue entry Dword 0 is defined in Figure 417. The
supported list of FDP Event Types shall be listed in ascending order of FDP Event Type.
Figure 417: FDP Events – Completion Queue Entry Dword 0
Bits Description
31:08 Reserved
07:00 Number of FDP Event Types (NOET) : This field specifies the number of FDP Event Types that are
contained in the data buffer (refer to Figure 420).

Figure 418: FDP Events – Command Dword 11
Bits Description
31:24 Reserved
23:16
Number of FDP Event Types (NOET):  This field specifies the number of FDP Event Types that are
contained in the data buffer (refer to Figure 420).
This field is ignored by the controller on a Get Features command.
15:00
Placement Handle (PHNDL):  This field specifies the Placement Handle associated with the Reclaim
Unit Handle affected by this command.
If the specified Placement Handle is not valid for the namespace, then the controller shall abort a Set
Feature command or a Get Feature command with a status code of Invalid Field in Command.

Figure 419: Flexible Data Placement – Command Dword 12
Bits Description
31:01 Reserved
00
FDP Event Enable (FDPEE): If this bit is set to ‘1’ , then FDP events are enabled on the Reclaim Unit
Handle associated with the Placement Handle. If this bit is cleared to ‘0’, then FDP events are disabled
on the Reclaim Unit Handle associated with the Placement Handle.
This field is ignored by the controller on a Get Features command.
This Feature shall be saveable (refer to section 4.4).
If:
• a Reclaim Unit Handle is shared by more than one namespace that exists in the same Endurance
Group; and
• a Set Features command specifies this Feature and one of those namespaces,
