---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 553
headings: ["8.1.15.3 Namespace Management Considerations for Exported NVM Subsystems (informative)", "8.1.16 Namespace Write Protection"]
---

# Source page 553

NVM Express® Base Specification, Revision 2.1

531
or
• via any other method (e.g., a method outside the scope of this specification or via a Management
Endpoint, refer to the NVM Express Management Interface Specification), then :
o each controller, to which the namespace is attached, that has Attached Namespace
Attribute Notices enabled (refer to Figure 391) shall issue an Attached Namespace
Attribute Changed asynchronous event as part of the delete operation to indicate a
namespace change; and
o each controller that has Allocated Namespace Attribute Notices enabled (refer to Figure
391) shall issue an Allocated Namespace Attribute Changed asynchronous event as part
of the delete operation to indicate a namespace change.
 Namespace Management Considerations for Exported NVM Subsystems (informative)
A host is able to determine whether an Exported NVM Subsystem supports the Namespace Attachment
command by reading the Commands Supported and Effects log page (refer to section 5.1.12.1.6). The
Namespace Attachment command is used to attach namespaces to controllers and detach namespaces
from controllers. The Namespace Management command is not supported by controllers in an Exported
NVM subsystem.
If an Exported Namespace is detached from a controller in the Exported NVM Subsystem, then the NSID
that referred to that namespace becomes an inactive NSID (refer to section 3.2.1.4) on that controller. If an
Underlying Namespace is disassociated from the Exported NVM Subsystem (refer to section 5.3.7.1.2),
then the NSID of the Exported Namespace that referred to that namespace becomes an unallocated NSID
(refer to section 3.2.1.3) and is not available to any controller in the Exported NVM Subsystem. Previously
submitted but uncompleted or subsequently submitted commands to the namespace that is:
• detached from a controller; or
• disassociated from the Exported NVM Subsystem,
are handled by the controller as if they were issued to an inactive NSID (refer to section 3.2.1.4 and Figure
92). Refer to section 8.1.15 for Host actions to attach or detach a specified Namespace.
If an Exported NVM Subsystem Exports an Underlying Namespace that becomes unavailable (e.g.,
detached or deleted) or is affected by a Capacity Management command (e.g., Endurance Group deletion,
NVM Set deletion (refer to section 8.1.4)) then:
• Previously submitted but uncompleted or subsequently submitted commands to the Exported
Namespace are handled by the controller as if they were issued to an inactive NSID (refer to section
3.2.1.4 and Figure 92); and
• If Namespace Attribute Notices are enabled controllers affected by Underlying Namespace
changes report a Namespace Attribute Changed asynchronous event to the host as described in
Figure 151.
 Namespace Write Protection
Namespace Write Protection is an optional configurable controller capability that enables the host to control
the write protection state of a namespace or to determine the write protection state of a namespace. Support
for this capability is reported in th e Namespace Write Protection Capabilities (NWPC) field in the Identify
Controller data structure (refer to Figure 312).
Figure 621 defines the write protection states that may be supported for a namespace. All states persist
across power cycles and Controller Level Resets (refer to section  3.7.2) except Write Protect Until Power
Cycle state, which transitions to the No Write Protect state on the occurrence of a power cycle.
Figure 621: Namespace Write Protection State Definitions
M/O State Definition
Persistent Across
Power
Cycles
Controller
Level Resets
M No Write Protect The namespace is not write protected. Yes Yes
