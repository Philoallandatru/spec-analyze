---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 568
headings: ["8.1.20 Read Recovery Level"]
---

# Source page 568

NVM Express® Base Specification, Revision 2.1

546
Namespace Data Structure for that namespace has changed), if RGIDC bit in the CRCAP field is
cleared to ‘0’ in the Identify Controller data structure (refer to Figure 312).
As a result of receiving a Reachability Group Change notice, the host should read the Reachability Groups
log page (refer to section 5.1.12.1.25) to check for each of those possible changes.
Receipt of a Reachability Association Change Notice from a controller may indicate:
a) an RGID has been added to one or more of the Reachability Association Descriptors;
b) an RGID has been removed from one or more of the Reachability Association Descriptors; and/or
c) a Reachability Association no longer has any RGIDs associated with this controller as members of
that Reachability Association and therefore is no longer reported in the Reachability Associations
log page for this controller.
As a result of receiving a Reachability Association Change notice, the host should read the Reachability
Associations log page (refer to section 5.1.12.1.26) to check for each of those possible changes.
 Read Recovery Level
The Read Recovery Level (RRL) is an NVM Set configurable attribute that balances the completion time
for read commands and the amount of error recovery applied to those read commands. The Read Recovery
Level applies to an NVM Set with which the Read Recovery Level is associated. A namespace  created
within an NVM Set inherits the Read Recovery Level of that NVM Set. If NVM Sets are not supported, all
namespaces in the NVM subsystem use an identical Read Recovery Level.
The controller indicates support for Read Recovery Levels in the Controller Attributes field in the Identify
Controller data structure (refer to Figure 312). If Read Recovery Levels are supported, then the specific
levels supported are indicated in the Read Recovery Levels Supported field in the Identify Controller data
structure. There are 16 levels that may be supported. Level 0, if supported, provides the maximum amount
of recovery. Level 4 is a mandatory level that provides a nominal amount of recovery and is the default
level. Level 15 is a mandatory level that provides the minimum amount of recovery and is referred to as the
‘Fast Fail’ level. The level s are organized based on the amount of recovery supported, such that a higher
numbered level provides less recovery than the preceding lower level.
Interactions between the Read Recovery Level and the Limited Retry (LR) field in I/O commands are
implementation specific.
The Read Recovery Level may be configured using a Set Features command for the Read Recovery Level
Config Feature. The Read Recovery Level may be determined using a Get Features command for the Read
Recovery Level Config Feature.
