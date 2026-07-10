---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 603
headings: ["8.1.24.5 Sanitize Operation Effects on Exported NVM Subsystems", "8.1.25 Submission Queue (SQ) Associations"]
---

# Source page 603

NVM Express® Base Specification, Revision 2.1

581
• any command or command option that is not explicitly permitted in Figure 142 shall be aborted with
a status code of Sanitize In Progress if processed by the controller;
• the Persistent Memory Region shall be prevented from being enabled (i.e., setting PMRCTL.EN to
‘1’ does not result in PMRSTS.NRDY being cleared to ‘0’); and
• activation of new firmware is prohibited.
While the sanitization target is in the Restricted Failure state or the Unrestricted Failure state, then for each
controller in the NVM subsystem:
• any command or command option that is not explicitly permitted in Figure 142 shall be aborted with
a status code of Sanitize In Progress if processed by the controller;
• all I/O commands other than a Flush command (refer to section 7.2), shall be aborted with a status
code of Sanitize Failed;
• the Sanitize command is permitted with action restrictions (refer to section 5.1.22); and
• the Persistent Memory Region shall be prevented from being enabled (i.e., setting PMRCTL.EN to
‘1’ does not result in PMRSTS.NRDY being cleared to ‘0’).
When a sanitize operation starts on any controller in an NVM subsystem (i.e., a transition into the Restricted
Processing state occurs or a transition into the Unrestricted Processing state occurs), all Management
Endpoints in the NVM subsystem shall perfo rm the sanitize operation as described in the NVM Express
Management Interface Specification.
 Sanitize Operation Effects on Exported NVM Subsystems
Performing a sanitize operation on an Underlying NVM Subsystem (refer to section  8.3.3) sanitizes user
data in all Underlying Namespaces contained in that NVM subsystem, including any Underlying Namespace
that is associated with an Exported Namespace in any Exported NVM Subsystem (refer to section 5.3.7).
If an Exported NVM Subsystem contains an Exported Namespace that is associated with an Underlying
Namespace in an Underlying NVM Subsystem for which one of the following conditions exists:
• a sanitize operation is in progress; or
• the most recent sanitize operation has failed and successful recovery from the failed sanitize
operation has not occurred,
then, while that condition exists, that Exported NVM Subsystem shall enforce the I/O command sanitize
operation restrictions described in section 8.1.24.4 on I/O commands that specify that Exported Namespace
and may enforce additional sanitize operation restrictions described in that section.
 Submission Queue (SQ) Associations
When Predictable Latency Mode is enabled, all I/O commands for namespaces in a given NVM Set have
the same quality of service attributes and shall exhibit predictable latencies as described in section 8.1.18.
The SQ Associations capability provides hints to the controller as to which specific I/O Queues are
associated with a given NVM Set. The controller uses this information to further enhance performance when
Predictable Latency Mode is enabled.
The SQ Associations capability is an optional capability. Predictable Latency Mode (refer to section 8.1.18)
is not dependent on the use of the SQ Associations capability.
If a controller supports SQ Associations, then the controller shall:
• indicate support for the SQ Associations capability in the Controller Attributes (CTRATT) field in
the Identify Controller data structure;
• indicate support for NVM Sets in the Controller Attributes (CTRATT) field in the Identify Controller
data structure; and
• indicate support for Predictable Latency Mode in the Controller Attributes (CTRATT) field in the
Identify Controller data structure (refer to Figure 312).
