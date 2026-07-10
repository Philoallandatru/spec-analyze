---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 395
headings: ["5.1.25.1 Common Feature Specific Information", "5.1.25.1.1 Arbitration (Feature Identifier 01h)"]
---

# Source page 395

NVM Express® Base Specification, Revision 2.1

373
Figure 385: Set Features – Feature Identifiers
Feature
Identifier
Current Setting
Persists
Across Power
Cycle and
Reset 2
Uses
Memory
Buffer for
Attributes
Feature Name Scope 6
C0h to FFh Vendor Specific1, 5
Notes:
1. The behavior of a controller in response to an inactive namespace ID to a vendor specific Feature Identifier is
vendor specific.
2. This column is only valid if the feature is not saveable (refer to section 4.4). If the feature is saveable, then this
column is not used.
3. The controller does not save settings for the Host Memory Buffer feature across power states and reset events,
however, host software may restore the previous values. Refer to section 8.2.3.
4. The feature does not use a memory buffer for Set Features commands and does use a memory buffer for Get
Features commands. Refer to section 5.1.25.2.4.
5. Selection of a UUID may be supported. Refer to section 8.1.28.
6. Refer to Feature Identifiers Supported and Effects log page in section 5.1.12.1.18 for how scope is reported to
the host.
7. For NVM Subsystems with multiple controllers in the same domain, specifying different power states results in
an unspecified power state for that domain.
 Common Feature Specific Information
Refer to section 3.1.3.6 for mandatory, optional, and prohibited features for the various controller types.
Some Features utilize a memory buffer to configure or return attributes for a Feature, whereas others only
utilize a dword in the command or completion queue entry . For more information on Features, including
default value definitions, saved value definitions, and current value definitions, refer to section 4.4.
Upon completion of a Set Features command for a feature, the host should rediscover, re-enumerate and/or
re-initialize all capabilities associated with that feature. For example, if a namespace capability change may
occur for a feature, then host software should pause the use of any associated namespace, submit the Set
Features command for that feature and wait for that command to complete, and then re -issue commands
to all namespaces affected by that Set Features command.
There may be commands in execution when a Feature is changed. The new settings may or may not apply
to commands already submitted for execution when the Feature is changed.  Any commands submitted to
a Submission Queue after a Set Features command is successfully completed shall utilize the new settings
for the associated Feature. To ensure that a Features values apply to all subsequent commands, the host
should allow commands being processed to complete prior to issuing the Set Features command.
If the controller does not support a changeable value for a Feature ( i.e., the Feature is not changeable ,
refer to section 5.1.11.2), and a Set Features command for that Feature is processed, then if that command
specifies a Feature value that:
• is not the same as the existing value for that Feature, then the controller shall abort that command
with a status code of Feature Not Changeable; and
• is the same as the existing value for that Feature, then the controller may:
o complete that command successfully; or
o abort that command with a status code of Feature Not Changeable.
Refer to each feature description in this section for any additional requirements associated with that feature.
5.1.25.1.1 Arbitration (Feature Identifier 01h)
This Feature controls command arbitration  within the controller . Refer to section 3.4.4 for command
arbitration details. The attributes are specified in Command Dword 11.
