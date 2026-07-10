---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 182
headings: []
---

# Source page 182

NVM Express® Base Specification, Revision 2.1

160
Figure 124: Current Value after Reset with Scope of Entire NVM Subsystem
Feature
Capability
Controller scope or
Namespace per
controller scope
NVM subsystem
scope
Endurance Group
scope
NVM Set
scope
Namespace
scope
Saveable
Set to:
• the saved value if a saved value is set; or
• the default value if a saved value is not set,
unless otherwise specified.
Non-saveable
and non-
persistent
Set to the default value, unless otherwise specified.
The current value for a Feature, as a result of a Controller Level Reset, is indicated in Figure 125 for an
NVM subsystem that is able to support two or more controllers (i.e., the Multiple Ports bit is set to ‘1’ in the
CMIC field).
The current value for a Feature, as a result of an NVM Subsystem Reset, is indicated in Figure 125 for an
NVM subsystem that supports:
• multiple domains where an NVM Subsystem Reset does not reset the entire NVM subsystem as
defined in section 3.7.1.2.
Figure 125: Current Value after Reset with Scope of Subset of the NVM Subsystem
Feature
Capability
Controller scope or Namespace per
controller scope
NVM
subsystem
scope
Endurance
Group scope
NVM Set
scope
Namespace
scope
Saveable
Set to:
• the saved value if a saved
value is set; or
• the default value if a saved
value is not set,
unless otherwise specified.
Unchanged, unless otherwise specified.
Non-saveable
and non-
persistent
Set to the default value,  unless
otherwise specified.
Concurrent access to Feature values by multiple hosts, for features with a scope other than controller
scope, requires some form of coordination between hosts. The procedure used to coordinate these hosts
is outside the scope of this specification.
Feature settings apply based on the feature scope (e.g., a feature is namespace specific if that feature has
a namespace scope) as defined in Figure 385.
For feature values that apply to the controller scope:
a) if the NSID field is cleared to 0h or set to FFFFFFFFh, then:
• the Set Features command shall set the specified feature value for the controller; and
• the Get Features command shall return the current setting of the requested feature value for
the controller;
and
b) if the NSID field is set to a valid namespace identifier (refer to section 3.2.1.2), then:
• the Set Features command shall abort with a status code of Feature Not Namespace Specific;
and
