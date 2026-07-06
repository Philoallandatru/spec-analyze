---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 170
headings: ["4.2.3.4 Path Related Status Definition"]
---

# Source page 170

NVM Express® Base Specification, Revision 2.1

148
Figure 106: Status Code – Media and Data Integrity Error Values
Value Description Command Set(s)1
00h to 7Fh Reserved
80h Write Fault: The write data could not be committed to the media.
81h Unrecovered Read Error: The read data could not be recovered from the
media.
82h End-to-end Guard Check Error: The command was aborted due to an end -
to-end guard check failure.
83h End-to-end Application Tag Check Error: The command was aborted due to
an end-to-end application tag check failure.
84h End-to-end Reference Tag Check Error: The command was aborted due to
an end-to-end reference tag check failure.
85h Compare Failure: See the NVM Command Set Specification for the
description. NVM
86h
Access Denied: Access to the namespace and/or user data is denied due to
lack of access rights. Refer to the appropriate security specification (e.g., TCG
Storage Interface Interactions Specification).

87h Deallocated or Unwritten Logical Block: See the NVM Command Set
Specification for the description.  NVM
88h End-to-End Storage Tag Check Error: The command was aborted due to an
end-to-end storage tag check failure.
89h to BFh Reserved
C0h to FFh Vendor Specific
Key:
NVM – NVM Command Set
Notes:
1. This column is blank unless the value is I/O Command Set specific.
 Path Related Status Definition
Completion queue entries with a Status Code Type (SCT) of Path Related Status (refer to Figure 107)
indicate a status value associated with the command that is generic across many different types of
commands and applies to a specific connection between the host and controller processing the command
or between the controller and the namespace. The command for which this status is returned may be retried
on a different controller in the same NVM subsystem if more than one controller is available to the host.
In a multipath environment, unless otherwise specified, errors of this type should be retried using a different
path, if one is available.
Figure 107: Status Code – Path Related Status Values
Value Description
00h
Internal Path Error:  The command was not completed as the result of a controller internal error
that is specific to the controller processing the command. Retries for the request function should be
based on the setting of the DNR bit (refer to Figure 100).
01h
Asymmetric Access Persistent Loss: The requested function (e.g., command) is not able to be
performed as a result of the relationship between the controller and the namespace, NVM Set, or
Endurance Group being in the ANA Persistent Loss state (refer to section 8.1.1.7). The command
should not be re-submitted to the same controller.
02h
Asymmetric Access Inaccessible:  The requested function (e.g., command) is not able to be
performed as a result of the relationship between the controller and the namespace, NVM Set, or
Endurance Group being in the ANA Inaccessible state (refer to section 8.1.1.6). The command
should not be re-submitted to the same controller.
03h
Asymmetric Access Transition:  The requested function (e.g., command) is not able to be
performed as a result of the relationship between the controller and the namespace, NVM Set, or
Endurance Group transitioning between Asymmetric Namespace Access states (refer to section
8.1.1.8). The requested function should be retried after the transition is complete.
