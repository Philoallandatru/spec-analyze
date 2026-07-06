---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 168
headings: []
---

# Source page 168

NVM Express® Base Specification, Revision 2.1

146
Figure 103: Status Code – Command Specific Status Values
Value Description Commands Affected
2Eh Namespace Is Dispersed Reservation Acquire, Reservation Register,
Reservation Release, Reservation Report
2Fh Invalid Discovery Information Discovery Information Management
30h Zoning Data Structure Locked Fabric Zoning Lookup, Fabric Zoning Send,
Fabric Zoning Receive
31h Zoning Data Structure Not Found Fabric Zoning Lookup, Fabric Zoning Send,
Fabric Zoning Receive
32h Insufficient Discovery Resources Discovery Information Management
33h Requested Function Disabled Fabric Zoning Lookup, Fabric Zoning Send,
Fabric Zoning Receive
34h ZoneGroup Originator Invalid Fabric Zoning Send
35h Invalid Host Manage Exported NVM Subsystem
36h Invalid NVM Subsystem  Manage Exported NVM Subsystem
37h Invalid Controller Data Queue Set Features, Get Features, Track Send,
Controller Data Queue
38h Not Enough Resources Track Send, Controller Data Queue
39h Controller Suspended Track Send, Sanitize
3Ah Controller Not Suspended Track Send
3Bh Controller Data Queue Full Track Send
3Ch to 6Fh Reserved
70h to 7Fh Directive Specific NOTE 1
80h to BFh I/O Command Set Specific Refer to Figure 104
C0h to FFh Vendor Specific
Notes:
1. The Directives Specific range defines Directives specific status values. Refer to section 8.1.8.

Figure 104: Status Code – Command Specific Status Values, I/O Commands
Value Description
80h Conflicting Attributes
81h Invalid Protection Information
82h Attempted Write to Read Only Range
83h Command Size Limit Exceeded
84h Invalid Command ID
85h Incompatible Namespace or Format
86h Fast Copy Not Possible
87h Overlapping I/O Range
88h Namespace Not Reachable
89h Insufficient Resources
8Ah Insufficient Program Resources
8Bh Invalid Memory Namespace
8Ch Invalid Memory Range Set
8Dh Invalid Memory Range Set Identifier
8Eh Invalid Program Data
8Fh Invalid Program Index
90h Invalid Program Type
91h Maximum Memory Ranges Exceeded
92h Maximum Memory Range Sets Exceeded
93h Maximum Programs Activated
94h Maximum Program Bytes Exceeded
95h Memory Range Set In Use
96h No Program
97h Overlapping Memory Ranges
