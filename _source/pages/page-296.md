---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 296
headings: ["5.1.12.1.29 Reclaim Unit Handle Usage (Log Page Identifier 21h)"]
---

# Source page 296

NVM Express® Base Specification, Revision 2.1

274
Figure 281: Reclaim Unit Handle Descriptor
Bytes Description
2h
Persistently Isolated: The user data from a write command that utilizes this
type of Reclaim Unit Handle is originally isolated in the referenced Reclaim
Unit from other user data in write commands that utilize a different Reclaim
Unit Handle. If the controller moves the user da ta due to vendor specific
operations (i.e., garbage collection), then the controller shall move that user
data to a Reclaim Unit in the same Reclaim Group that only contains user data
that was written by the host utilizing the same Reclaim Unit Handle. Ref er to
section 8.1.10.
3h to BFh Reserved
C0h to FFh Vendor Specific

03:01 Reserved
The format of the Placement Identifier used in the Data Placement Directive (refer to section 8.1.8.4) for a
specific FDP configuration is dependent on the number of Reclaim Groups supported by the FDP
configuration in use and the number of bits allocated for the Reclaim Group Identifier field. If the NRG field
(refer to Figure 280) is set to 1h indicating that there is only one Reclaim Group and zero bits are allocated
to the Reclaim Group Identifier (i.e., the RGIF field (refer to Figure 280) is cleared to 0h), then the format
of the Placement Identifier is defined in Figure 282. If one or more bits are allocated to the Reclaim Group
Identifier (i.e., the RGIF field is non -zero), then the format of the Placement Identifier is defined in Figure
283.
Figure 282: Placement Identifier Format without Reclaim Group Identifier
Bits Description
15:00 Placement Handle  (PHNDL): This field specifies a Placement Handle associated with the
namespace that is used by the controller to determine the Reclaim Unit Handle.

Figure 283: Placement Identifier Format with a non-zero RGIF
Bits Description
15:(16-RGIF)
Reclaim Group Identifier (RGID):  This field specifies the Reclaim Group used by the write
command.
If the NRG field (refer to Figure 280) is set to 1h (i.e., there is only one Reclaim Group), then
this field shall be ignored by the controller.
(15-RGIF):00 Placement Handle (PHNDL):  This field specifies a Placement Handle associated with the
namespace that is used by the controller to determine the Reclaim Unit Handle.
5.1.12.1.29 Reclaim Unit Handle Usage (Log Page Identifier 21h)
The Reclaim Unit Handle Usage log page (refer to Figure 284) is used to provide information about the
Reclaim Unit Handles associated with the Placement Handles of the namespaces in the specified
Endurance Group. The Endurance Group is specified by the Endurance Group Identifier field in the Log
Specific Identifier field as defined in Figure 217).
If Flexible Data Placement is enabled in the Endurance Group specified by the Endurance Group Identifier
field and a Get Log Page command specifies the Reclaim Unit Handle Usage log page, then the NSID field
in that command is reserved.
If Flexible Data Placement is disabled in the specified Endurance Group and the Get Log Page command
specifying this log page, then the controller shall abort the command with a status code of FDP Disabled.
