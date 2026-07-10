---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 319
headings: []
---

# Source page 319

NVM Express® Base Specification, Revision 2.1

297
Section 5.1.13.2 describes Identify data structures  that are common to all transport models . Section
5.1.13.3 describes Identify data structures that are specific to the Memory-based transport model. Section
5.1.13.4 describes Identify data structures that are specific to the Message-based transport model.
Figure 310: Identify – CNS Values
CNS
Value O/M1 Definition NSID2 CNTID3 CSI4 Reference
Section
Active Namespace Management
00h M11
Identify Namespace data structure for the specified
NSID or the namespace capabilities for the NVM
Command Set.7
Y N N8
NVM Command
Set
Specification
01h M
Identify Controller data structure for the controller
processing the command.7 N N N 5.1.13.2.1
02h M Active Namespace ID list. Y N N 5.1.13.2.2
03h M Namespace Identification Descriptor list for the
specified NSID. Y N N 5.1.13.2.3
04h O
An NVM Set List (refer to Figure 317) is returned to
the host for up to 31 NVM Sets. The list contains
entries for NVM Set identifiers greater than or equal
to the value specified in the NVM Set Identifier
(CDW11.NVMSETID) field.
N N N 5.1.13.2.4
05h M
I/O Command Set specific Identify Namespace data
structure for the specified NSID for the I/O Command
Set specified in the CSI field.7
Y N Y 5.1.13.2.5
06h M
I/O Command Set specific Identify Controller data
structure for the controller processing the command.7 N N Y 5.1.13.2.6
07h M Active Namespace ID list associated with the
specified I/O Command Set. Y N Y 5.1.13.2.7
08h M I/O Command Set Independent Identify Namespace
data structure. Y N N 5.1.13.2.8
09h O
Identify Namespace data structure for the specified
Format Index containing the namespace capabilities
for the NVM Command Set.7
N N Y
NVM Command
Set
Specification
0Ah O
I/O Command Set specific Identify Namespace data
structure for the specified Format Index containing
the namespace capabilities for the I/O Command Set
specified in the CSI field.7
N N Y
I/O Command
Set
Specification
0Bh to 0Fh  Reserved
Controller and Namespace Management
10h O5 Allocated Namespace ID list. Y N N 5.1.13.2.9
11h O5, 11 Identify Namespace data structure for the specified
allocated NSID. Y N N8 5.1.13.2.10
12h O5 Controller List of controllers attached to the specified
NSID. Y Y N 5.1.13.2.11
13h O5 Controller List of controllers that exist in the NVM
subsystem. N Y N 5.1.13.2.12
14h O6 Primary Controller Capabilities data structure for the
specified primary controller. N Y N 5.1.13.3.1
15h O6 Secondary Controller list of controllers associated
with the primary controller processing the command. N Y N 5.1.13.3.2
16h O11 A Namespace Granularity List (refer to the NVM
Command Set Specification) is returned to the host
for up to sixteen Namespace Granularity Entries.
N N N8 5.1.13.2.13
17h O A UUID List (refer to Figure 320) is returned to the
host. N N N 5.1.13.2.14
