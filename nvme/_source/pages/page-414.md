---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 414
headings: ["5.1.25.1.22 Namespace Admin Label (Feature Identifier 1Fh)"]
---

# Source page 414

NVM Express® Base Specification, Revision 2.1

392
then a modification to this Feature occurs to that shared Reclaim Unit Handle and the Placement Handle
that is associated with the shared Reclaim Unit Handle.
Figure 420: FDP Events – Set Feature Data Structure
Bytes Description
FDP Event Type List
00 FDP Event Type 0: This field contains the first FDP Event Type (refer to Figure 289).
01 FDP Event Type 1: This field contains the second FDP Event Type.
…
NOET-1 FDP Event Type NOET-1: This field contains the last FDP Event Type.

Figure 421: FDP Events – Get Feature Data Structure
Bytes Description
Supported FDP Event Type List
01:00 Supported FDP Event Type 0:  This field contains the Supported FDP Event Descriptor for the
first Supported FDP Event Type.
03:02 Supported FDP Event Type 1:  This field contains the Supported FDP Event Descriptor for the
second Supported FDP Event Type.
…
(NOET-1)*2+1:
(NOET-1)*2
Supported FDP Event Type NOET -1: This field contains the Supported FDP Event Descriptor
for the last Supported FDP Event Type.

Figure 422: Supported FDP Event Descriptor
Bytes Description
00 FDP Event Type (FDPET): This field contains the FDP Event Type (refer to Figure 289).
01
FDP Event Type Attributes (FDPETA): This field contains attributes for the FDP Event Type.
Bits Description
07:01 Reserved
00 FDP Event Enable (FDPEE): If this bit is set to ‘1’ then this FDP Event Type is enabled. If
this bit is cleared to ‘0’, then this FDP Event Type is disabled.

5.1.25.1.22 Namespace Admin Label (Feature Identifier 1Fh)
The Namespace Admin Label feature provides the ability to set and get the Namespace Admin Label for a
namespace. This Feature shall be saveable and therefore shall not be supported if  the Save and Select
Feature Support (SSFS)  bit is cleared to ‘0’ in the Optional NVM Command Support (ONCS) field of the
Identify Controller data structure (refer to Figure 312). The attributes in Figure 423 are transferred in the
data buffer.
The saved value and current value of this Feature shall be identical. If the SV bit is cleared to ‘0’ in a Set
Features command that specifies this Feature, the controller shall abort the command with a status code
of Invalid Field in the Command. If the value of the Namespace Admin Label is changed by means outside
the scope of this standard, then that change shall affect the results of any subsequent Get Features
command that specifies this Feature.
The default value of this Feature is all nulls (i.e., all bytes cleared to 0h). Sanitize operations (refer to section
8.1.24) affect the values of this Feature; any successful sanitize operation shall modify this Feature by
resetting both the saved value and the current value to the default value.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 423 are returned
in the data buffer for that command.
