---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 550
headings: ["8.1.15 Namespace Management"]
---

# Source page 550

NVM Express® Base Specification, Revision 2.1

528
The address of a management agent contained in an embedded management controller (e.g., a BMC) is
set and retrieved using the Embedded Management Controller Address feature (refer to section
5.1.25.1.24). The Embedded Management Controller Address feature is intended to be set only by the
storage system containing the embedded management controller. The embedded management controller
is able to ensure this by issuing a Lockdown command (refer to section 5.1.15) with the OFI field specifying
this Feature, and the PRHBT bit set to ‘1’ and the IFC field set to Admin Submission Queue (i.e., 00b).
The address of a management agent contained in host software is set and retrieved using the Host
Management Controller Address feature (refer to section 5.1.25.1.25). The Host Management Agent
Address feature is intended to be set only by host software. Host software is able to ensure this by issuing
a Lockdown command (refer to section 5.1.15) with the OFI field specifying this Feature, and the PRHBT
bit set to ‘1’ and the IFC field set to Out-of-band on a Management Endpoint (i.e., 10b).
 Namespace Management
The Namespace Management capability consist s of the Namespace Management command  (refer to
section 5.1.21) and the Namespace Attachment command  (refer to section 5.1.20). The Namespace
Management command is used to create a namespace or delete a namespace. The Namespace
Attachment command is used to attach and detach controllers from a namespace. The Namespace
Management capability is intended for use during manufacturing or by a system administrator.
In addition to the Namespace Management capability, there are other events and capabilities that are able
to affect the number of namespaces that exist within an NVM subsystem (e.g., the Capacity Management
capability as described in section 8.1.4).
If the Namespace Management capability is supported, then the controller:
a) shall support the Namespace Management command and the Namespace Attachment
command;
b) shall set the NMS bit to ‘1’ in the OACS field (refer to Figure 312);
c) shall support the Attached Namespace Attribute Changed asynchronous event (refer to Figure
151 and section 5.1.25.1.5);
d) should support the Allocated Namespace Attribute Changed asynchronous event ( refer to
Figure 151 and section 5.1.25.1.5); and
e) may support Namespace Granularity (refer to the NVM Command Set Specification).
A controller may support the Namespace Attachment command without supporting the Namespace
Management command. Such a controller:
a) shall not set the Namespace Management Supported (NMS)  bit to ‘1’ in the OACS field (refer
to Figure 312); and
b) should support the Namespace Attribute Change asynchronous event.
If a namespace is detached from a controller, then the NSID that referred to that namespace becomes an
inactive NSID (refer to section 3.2.1.4) on that controller. If a namespace is deleted from the NVM
subsystem, then the NSID that referred to that namespace becomes an unallocated NSID (refer to section
3.2.1.3) in the NVM subsystem. Previously submitted but uncompleted or subsequently submitted
commands to the affected NSID are handled by the controller as if they were issued to an inactive NSID
(refer to Figure 92).
The size of a namespace that contains formatted storage  is based on the size requested in a create
operation, the format of the namespace, and any characteristics (e.g., endurance). The controller
determines the NVM capacity allocated for that namespace. Namespaces may be created with different
usage characteristics (e.g., enduran ce) that utilize differing amounts of NVM capacity. Namespace
characteristics and the mapping of these characteristics to NVM capacity usage are outside the scope of
this specification.
Reporting of capacity information for the NVM subsystem, Domain, Endurance Group, and NVM Set are
described in section 3.8. For each namespace  that contains formatted storage , the NVM Set and the
Endurance Group that contain the namespace are reported in the Identify Namespace data structure. The
NVM Set to be used for a namespace  that contains formatted storage  is based on the value in the NVM
