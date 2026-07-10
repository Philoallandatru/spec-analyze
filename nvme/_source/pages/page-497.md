---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 497
headings: ["8 Extended Capabilities", "8.1 Common Extended Capabilities", "8.1.1 Asymmetric Namespace Access Reporting", "8.1.1.1 Asymmetric Namespace Access Reporting Overview"]
---

# Source page 497

NVM Express® Base Specification, Revision 2.1

475
8 Extended Capabilities
This section describes extended capabilities that are optional. Section 8.1 describes extended capabilities
that are common to all transport models . Section 8.2 describes extended capabilities  that are specific to
the Memory-based transport model . Section 8.3 describes extended capabilities  that are specific to the
Message-based transport model.
8.1 Common Extended Capabilities
This section describes extended capabilities that are common to all transport models.
 Asymmetric Namespace Access Reporting
 Asymmetric Namespace Access Reporting Overview
Asymmetric Namespace Access (ANA) occurs in environments where namespace access characteristics
(e.g., performance or ability to access the media) may vary based on:
• the controller used to access the namespace (e.g., Fabrics); and
• the internal configuration of the NVM subsystem. Asymmetric Namespace Access Reporting is
used to indicate to the host information about those access characteristics.
Shared namespaces may be accessed through controllers via multiple PCIe ports or fabric ports (refer to
section 2.4.1). The controllers that provide access to a shared namespace may provide identical access
characteristics through all controllers (i.e., symmetric access), or may provide different access
characteristics through some controllers (i.e., asymmetric access).
Private namespaces are accessed by only one controller at a time. The access characteristics of the
namespace through that controller may be impacted as a result of changes to the internal configuration of
the NVM subsystem. If the access characteristics o f the namespace through that controller are impacted
by the internal configuration of the NVM subsystem, then asymmetric access occurs.
Symmetric access to a namespace occurs when:
• the access characteristics using one controller are identical to the access characteristics when
using a different controller; and
• changes to the internal configuration of the NVM subsystem do not impact the access
characteristics.
Asymmetric access to a namespace occurs when:
• the access characteristics using one controller may differ from the access characteristics when
using a different controller; or
• changes to the internal configuration of the NVM subsystem may impact the access characteristics.
While commands may be sent to a shared namespace through any attached controller with asymmetric
access, the characteristics (e.g., performance or ability to access the media) may differ based on which
controller is used; as a result, the host should consider those characteristics when selecting which controller
to use for each command that accesses the namespace. The NVM subsystem may perform autonomous
internal reconfiguration that results in a change to the access characteristics.
If an NVM subsystem supports Asymmetric Namespace Access Reporting, then all controllers in that NVM
subsystem shall:
• set the Asymmetric Namespace Access Reporting Support (ANARS) bit to ‘1’ in the Controller
Multi-path I/O and Namespace Sharing Capabilities (CMIC) field in the Identify Controller data
structure (refer to Figure 312) to indicate support for Asymmetric Namespace Access Reporting;
• set the Report ANA Optimized State (RANAOS) bit  to ‘1’ in the Asymmetric Namespace Access
Capabilities (ANACAP) field in the Identify Controller data structure to indicate that the ANA
Optimized state is able to be reported;
