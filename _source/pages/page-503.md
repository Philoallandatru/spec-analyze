---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 503
headings: ["8.1.2 Asymmetric Namespace Access Reporting – Host Considerations (Informative)", "8.1.2.1 Host ANA Normal Operation"]
---

# Source page 503

NVM Express® Base Specification, Revision 2.1

481
Figure 588: ANA effects on Command Processing
Command ANA State Effects on command processing
Identify ANA Inaccessible or
ANA Persistent Loss
Capacity fields in the Identify Namespace data structure (refer to
the applicable I/O Command Set specification) information are
cleared to 0h.
Set Features
ANA Inaccessible
The saving of features shall not be supported and the following
feature identifiers are not available1:
a) Reservation Notification Mask (i.e., 82h);
b) Reservation Persistence (i.e., 83h); and
I/O Command Set specific feature identifiers2.
If the NSID is set to FFFFFFFFh, then the command shall abort3
with a status code of Asymmetric Access Inaccessible (refer to
section 8.1.1.6).
ANA Change
The saving of features shall not be supported and the following
feature identifiers are not available1:
a) Reservation Notification Mask (i.e., 82h);
b) Reservation Persistence (i.e., 83h); and
c) I/O Command Set specific feature identifiers2.
If the NSID is set to FFFFFFFFh, then the command shall abort3
with a status code of Asymmetric Access Transition (refer to section
8.1.1.8).
ANA Persistent Loss The command shall abort with a status code of Asymmetric Access
Persistent Loss (refer to section 8.1.1.7).
Notes:
1. If the ANA state is ANA Inaccessible State, then commands that use feature identifiers or log pages that are not
available shall abort with a status code of Asymmetric Access Inaccessible. If the ANA state is ANA Persistent
Loss State, then commands that use feature identifiers or log pages that are not available shall abort with a
status code of Asymmetric Access Persistent Loss. If the ANA state is ANA Change State, then commands that
use feature identifiers or log pages that are not available shall abort with a status code of Asymmetric Access
Transition.
2. I/O Command Set specific definition. Refer to each I/O Command Set specification for applicability and additional
details, if any.
3. If any namespace that is attached to the controller is in an ANA Group that is in the ANA Inaccessible state, the
ANA Persistent Loss state, or the ANA Change state, then the command shall abort with the indicated status.
Depending on the ANA state of the ANA Group that contains a namespace (e.g., an ANA state changes during
the processing of the command), the specified feature identifier may be altered for some attached namespaces
and not altered for other attached namespaces.
 Asymmetric Namespace Access Reporting – Host Considerations (Informative)
 Host ANA Normal Operation
The host determines if ANA is supported by examining the Asymmetric Namespace Access Reporting
Support (ANARS) bit in the CMIC field in the Identify Controller data structure (refer to Figure 312). The
NSID or identifier (refer to section 4.5) is used to determine when multiple paths to the same namespace
that is not a dispersed namespace are available. For dispersed namespaces, globally unique namespace
identifiers (refer to section 8.1.9.2) are used to determine when multiple paths are available to the same
namespace, as described in section 8.1.9.5. The host examines the ANA log page (refer to section
5.1.12.1.13) for each controller to determine the ANA state of each group of namespaces attached to that
controller.
To send a command to a namespace, the host should select a controller that reports the ANA Optimized
State (refer to section 8.1.1.4) and send the command to that controller. If more than one controller that
