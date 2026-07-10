---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 35
headings: ["1.5.85 rotational media", "1.5.86 Runtime D3 (Power Removed)", "1.5.87 sanitize operation", "1.5.88 sanitization target", "1.5.89 secondary controller", "1.5.90 shared namespace", "1.5.91 specified namespace", "1.5.92 spindown", "1.5.93 spinup", "1.5.94 static controller", "1.5.95 Underlying Namespace", "1.5.96 Underlying Namespace List"]
---

# Source page 35

NVM Express® Base Specification, Revision 2.1

13
 rotational media
Media that stores data on rotating platters (refer to section 8.1.23).
 Runtime D3 (Power Removed)
In Runtime D3 (RTD3) main power is removed from the controller. Auxiliary power may or may not be
provided. For PCI Express, RTD3 is the D3cold power state (refer to section 8.1.17.4).
 sanitize operation
Process by which all user data in the NVM subsystem is altered such that recovery of the previous user
data from any cache or the non-volatile storage media is infeasible for a given level of effort (refer to IEEE
2883™-2022).
 sanitization target
The target of a sanitize operation (i.e., an NVM subsystem).
 secondary controller
An NVM Express controller that depends on a primary controller in an NVM subsystem for management of
some controller resources (refer to section 8.2.6).
A PCI Express SR-IOV Virtual Function that supports the NVM Express interface and receives resources
from a primary controller is an example of a secondary controller (refer to section 8.2.6.4).
 shared namespace
A namespace that may be attached to two or more controllers in an NVM subsystem concurrently. Refer to
the Namespace Multi-path I/O and Namespace Sharing Capabilities (NMIC) field in Figure 319.
 specified namespace
The namespace that is associated with the value specified by the Namespace Identifier (NSID) field in a
command as defined by the Common Command Format (refer to Figure 92).
 spindown
The process of ch anging a spindle from an operational power state to a non -operational power state, for
an Endurance Group that stores data on rotational media (refer to section 8.1.23).
 spinup
The process of changing a spindle from a non -operational power state to an operational power state, for
an Endurance Group associated with rotational media (refer to section 8.1.23).
 static controller
The controller is pre-existing with a specific Controller ID and its state (e.g., Feature settings) is preserved
from prior associations.
 Underlying Namespace
A namespace (defined in section 1.5.62) accessible through physical or virtual functions in an Underlying
NVM Subsystem that may be used to associate with an Exported NVM Subsystem. Underlying
Namespaces are identified by the Underlying Namespace Entry data structure (refer to Figure 334).
 Underlying Namespace List
A list of namespaces (refer to section 5.1.13.4.1) in all underlying NVM subsystems that may be used to
create an Exported Namespace.
