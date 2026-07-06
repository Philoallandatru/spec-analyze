---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 105
headings: []
---

# Source page 105

NVM Express® Base Specification, Revision 2.1

83
A domain is comprised of:
• zero or more controllers; and
• zero or more NVM Endurance Groups.
If an NVM subsystem supports multiple domains, then all controllers in that NVM subsystem shall:
• set the MDS bit to ‘1’ in the CTRATT field in the Identify Controller data structure (refer to  Figure
312);
• set the Domain Identifier in each Endurance Group descriptor, if supported, to a non -zero value;
and
• set the Domain Identifier in each Identify Controller data structure to a non-zero value.
If an NVM subsystem supports multiple domains, then controllers in that NVM subsystem may:
• support Endurance Groups (refer to Endurance Groups bit in the CTRATT field of Identify Controller
data structure).
For an NVM subsystem that supports multiple domains, each domain shall be assigned a domain identifier
that is unique within the NVM subsystem (refer to the Domain Identifier field in Figure 312 and section
3.2.5.3). For an NVM subsystem that do es not support multiple domains, Domain Identifier fields are
cleared to 0h.
Figure 71 shows an example of an NVM subsystem that consists of three domains. Domain 1 contains two
controllers and some amount of NVM storage capacity which has been allocated to two private namespaces
(i.e., NS A and NS C) and a shared namespace (i.e., NS B). Dom ain 2 contains two controllers and some
amount of NVM storage capacity which has been allocated to two shared namespaces (i.e., NS D and NS
E). Domain 3 contains one controller, and no NVM storage capacity.
Figure 71: Example 1 Domain Structure
NSID 1 NSID 2
PCI Function 0
NVMe Controller
NS
A
NS
B
NS
C
Port v Port w
PCI Function 0
NVMe Controller
PCI Function 0
NVMe Controller
PCI Function 0
NVMe Controller
PCI Function 0
NVMe Controller
Port x Port y Port z
Domain 1 Domain 2 Domain 3
Communication
boundary
Communication
boundary
NSID 3 NSID 2 NSID 2NSID 2 NSID 2NSID 4 NSID 4NSID 4
NS
D
Communication
boundary
Communication
boundary
To Domain 1To Domain 3
NS
E

If, in the example shown in Figure 71, a division event occurs that results in Domain 1 no longer being able
to communicate with Domain 2 and Domain 3, then the NVM subsystem would consist of two parts. The
first part consists of Domain 1 and the second part consists of Domain 2 and Domain 3.
Figure 72 shows an example of an NVM subsystem that consists of six domains, of which, three are
domains that contain controllers. Domain 1 is a domain that contains two controllers and some amount of
NVM storage capacity from which NVM Endurance Groups have been cr eated that contain a private
namespace (i.e., NS A) and a shared namespace (i.e., NS C ). Domain 2 is a domain that contains no
controllers and contains some amount of NVM storage capacity from which NVM Endurance Groups have
been created that contain a shared namespace (i.e., NS B). Domain 3 is a domain that contains two
