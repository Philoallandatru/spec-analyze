---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 465
headings: []
---

# Source page 465

NVM Express® Base Specification, Revision 2.1

443
Figure 523: Subsystem Management Data Structure
Bytes Description
63:00 Reserved
65:64
Number of Host Entries (NUMHENT): Specifies the number of Host Entries in this
data structure. This field shall be greater than 0h.  The value of this field is
represented as M.
67:66
Number of Exported NVM Subsystem Entries (NUMENSE): Specifies the number
of Exported NVM Subsystem Entries in this data structure. This field shall be greater
than 0h. The value of this field is represented as N.
255:68 Reserved
Host Entry List
575:256 Host Entry 1: Contains a Host Entry data structure as defined by Figure 524.
895:576 Host Entry 2: Contains a Host Entry data structure as defined by Figure 524, if any.
… …
320*M+255:320*(M-1)+256 Host Entry M: Contains a Host Entry data structure as defined by Figure 524, if any.
Exported NVM Subsystem Entry List
320+(320*M+255):
320*(N-1)+(320*M+256)
Exported NVM Subsystem Entry 1: Contains an Exported NVM Subsystem Entry
data structure as defined by Figure 525.
… …
320*N+(320*M+255):
320*(N-1)+(320*M+256)
Exported NVM Subsystem Entry N: Contains an Exported NVM Subsystem Entry
data structure as defined by Figure 525, if any.

Figure 524: Host Entry Data Structure
Bytes Description
07:00 Reserved
23:08 Host Identifier (HOSTID): This field has the same definition as the Host Identifier defined in Figure 438.
279:24 Host NVMe Qualified Name (HOSTNQN):  NVMe Qualified Name (NQN) that uniquely identifies the
host. Refer to section 4.7.
319:280 Reserved

Figure 525: Exported NVM Subsystem Entry Data Structure
Bytes Description
23:00 Reserved
279:24
NVM Subsystem NVMe Qualified Name (SUBNQN):  Uniquely identifies an Exported NVM
Subsystem. The Allowed Host List associated to this Exported NVM Subsystem is being modified. Refer
to section 4.7.
281:280 Port ID of the Underlying Port (PIDUP): Refer to the PORTID field in Figure 294.
319:282 Reserved
The controller shall abort the command with a status code of Invalid Field in Command if:
• the Number of Host Entries field in  the Subsystem Management Data Structure (refer to  Figure
523) is cleared to 0h; or
• the Number of Exported NVM Subsystem Entries field in the Subsystem Management Data
Structure (refer to Figure 523) is cleared to 0h.
Command specific status values associated with the Grant Host Access operation of the Manage Exported
NVM Subsystem command are defined in Figure 526. For failures, the byte offset of a failing entry shall be
reported in the Command Specific Information field of the Error Information Log Entry.
