---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 99
headings: ["3.2.1.7 I/O Command Set Associations", "3.2.2 NVM Sets"]
---

# Source page 99

NVM Express® Base Specification, Revision 2.1

77
1. Issue an Identify command with the CNS field cleared to 0h for each valid NSID (based on the
Number of Namespaces value (i.e., MNAM field or NN field) in the Identify Controller data
structure). If a non-zero data structure is returned for a particular NSID, then that is an active NSID;
or
2. Issue an Identify command with a CNS field set to 2h to retrieve a list of up to 1,024 active NSIDs.
If there are more than 1,024 active NSIDs, continue to issue Identify commands with a CNS field
set to 2h until all active NSIDs are retrieved.
To determine the allocated NSIDs in the NVM subsystem, the host may issue an Identify command with
the CNS field set to 10h to retrieve a list of up to 1,024 allocated NSIDs. If there are more than 1,024
allocated NSIDs, continue to issue Identify commands with a CNS field set to 10h until all allocated NSIDs
are retrieved.
Namespace IDs may change across power off conditions. However, it is recommended that namespace
IDs remain static across power off conditions to avoid issues with host software. To determine if the same
namespace has been encountered, the host may use the:
a) UUID field in the Namespace Identification Descriptor (refer to Figure 315), if present;
b) NGUID field in the Identify Namespace data (refer to the applicable I/O Command Set specification)
or in the Namespace Identification Descriptor, if present; or
c) EUI64 field in the Identify Namespace data or in the Namespace Identification Descriptor, if present.
UIDREUSE bit in the NSFEAT field (refer to Figure 319 or the Identify Namespace data structure in the
NVM Command Set Specification, if applicable) indicates NGUID and EUI64 reuse characteristics.
If Asymmetric Namespace Access Reporting is supported  (i.e., the Asymmetric Namespace Access
Reporting Support (ANARS) bit is set to ‘1’ in the CMIC field in the Identify Controller data structure (refer
to Figure 312)), refer to the applicable I/O Command Set specification for additional detail, if any.
A namespace may or may not have a relationship to a Submission Queue; this relationship is determined
by the host software implementation. The controller shall support access to any attached namespace from
any I/O Submission Queue.
 I/O Command Set Associations
A namespace is associated with exactly one I/O Command Set. For I/O commands and I/O Command Set
specific Admin commands, the I/O Command Set with which a submission queue entry is associated is
determined by the Namespace Identifier (NSID) field in the command.
An NVM subsystem may contain namespaces each of which is associated with a different I/O Command
Set. A controller may support attached namespaces that use any of the I/O Command Sets that the
controller simultaneously supports as indicated in the I/O Comm and Set Profile (refer to section
5.1.25.1.17).
 NVM Sets
An NVM Set is a collection of NVM that is separate (logically and potentially physically) from NVM in other
NVM Sets. One or more namespaces  that contain formatted storage  may be created within an NVM Set
and those namespaces inherit the attributes of the NVM Set. A namespace that contains formatted storage
is wholly contained within a single NVM Set and shall not span more than one NVM Set.
Figure 67 shows an example of three NVM Sets. NVM Set A contains three namespaces (NS A1, NS A2,
and NS A3). NVM Set B contains two namespaces (NS B1 and NS B2). NVM Set C contains one
namespace (NS C1). Each NVM Set shown also contains ‘Unallocated’ regions that consist of NVM that is
not yet allocated to a namespace.
