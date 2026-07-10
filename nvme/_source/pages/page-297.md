---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 297
headings: ["5.1.12.1.30 Flexible Data Placement (FDP) Statistics (Log Page Identifier 22h)"]
---

# Source page 297

NVM Express® Base Specification, Revision 2.1

275
Figure 284: Reclaim Unit Handle Usage Log Page
Bytes Description
Header
01:00
Number of Reclaim Unit Handles (NRUH) : This field identifies the number of Reclaim Unit
Handle Usage Descriptors in the Reclaim Unit Handle Usage Descriptor List. This field shall be
a non-zero value.
This field shall be set to the value of the NRUH field in the current FDP configuration (refer to
Figure 280) for the specified Endurance Group.
07:02 Reserved
Reclaim Unit Handle Usage Descriptor List
15:08 Reclaim Unit Handle Usage Descriptor  0: Contains the Reclaim Unit Handle Usage
Descriptor (refer to Figure 285) for Reclaim Unit Handle 0.
23:16 Reclaim Unit Handle Usage Descriptor 1:  Contains the Reclaim Unit Handle Usage
Descriptor (refer to Figure 285) for Reclaim Unit Handle 1.
…
(NRUH-1)*8+15:
(NRUH-1)*8+8
Reclaim Unit Handle Usage Descriptor NRUH -1: Contains the Reclaim Unit Handle Usage
Descriptor (refer to Figure 285) for Reclaim Unit Handle NRUH-1.
The Reclaim Unit Handle Usage Descriptor is defined in Figure 285.
Figure 285: Reclaim Unit Handle Usage Descriptor
Bytes Description
00
Reclaim Unit Handle Attributes (RUHA): This field identifies attributes associated with the
Reclaim Unit Handle Usage.
Value Definition
0h Not used by a namespace.
1h
Host Specified: The Reclaim Unit Handle Identifier is used by a namespace
created with a Namespace Management command explicitly requesting the
usage of this Reclaim Unit Handle (i.e., specifying a Placement Handle List).
2h
Controller Specified:  The Reclaim Unit Handle Identifier is used by a
namespace created with a Namespace Management command requesting the
controller to select the one and only Reclaim Unit Handle (i.e., not specifying a
Placement Handle List).
3h to FFh Reserved

07:01 Reserved
In the list of Reclaim Unit Handle Usage Descriptors, no more than one entry shall have the Reclaim Unit
Handle Attributes field set to Controller Specified (i.e., 2h).
5.1.12.1.30 Flexible Data Placement (FDP) Statistics (Log Page Identifier 22h)
The FDP Statistics log page (refer to Figure 286) is used to provide information about the FDP configuration
over the life of the FDP configuration in an Endurance Group. The Endurance Group is specified by the
Endurance Group Identifier field in the Log Specific Identifier field as defined in Figure 217).
If Flexible Data Placement is enabled in the specified Endurance Group and the Get Log Page command
specifies this log page, then the NSID field in that command is reserved.
If Flexible Data Placement is disabled in the specified Endurance Group and the Get Log Page command
specifies this log page, then the controller shall abort the command with a status code of FDP Disabled.
