---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 602
headings: ["8.1.24.4 Sanitize Operation Restrictions"]
---

# Source page 602

NVM Express® Base Specification, Revision 2.1

580
Transition Post-Verification Deallocation:Idle:
The controller shall:
a) report the Sanitize Operation Completed asynchronous event as described in section  8.1.24.2.
Transition Post-Verification Deallocation:Restricted Failure:
The controller shall:
a) report the Sanitize Operation Completed asynchronous event as described in section 8.1.24.2; and
b) set the FAILS field to 6h (i.e., Post-Verification Deallocation state).
Transition Post-Verification Deallocation:Unrestricted Failure:
The controller shall:
a) report the Sanitize Operation Completed asynchronous event as described in section 8.1.24.2; and
b) set the FAILS field to 6h (i.e., Post-Verification Deallocation state).
 Sanitize Operation Restrictions
In the following states:
• Restricted Processing;
• Restricted Failure;
• Unrestricted Processing;
• Unrestricted Failure;
• Media Verification; and
• Post-Verification Deallocation,
all enabled controllers in the NVM subsystem are restricted to performing only a limited set of actions.
When a sanitize operation starts on any controller (i.e., a transition into the Restricted Processing state
occurs or a transition into the Unrestricted Processing state occurs), all controllers in the NVM subsystem
shall:
• clear all of the following outstanding asynchronous events:
o Sanitize Operation Completed asynchronous event, if any;
o Sanitize Operation Completed With Unexpected Deallocation asynchronous event, if any;
and
o Sanitize Operation Entered Media Verification State asynchronous event, if any;
• update the Sanitize Status log page (refer to section 5.1.12.1.33);
• abort any command (submitted or in progress) not allowed during a sanitize operation (refer to
Figure 142) with a status code of Sanitize In Progress, unless otherwise specified;
• abort device self-test operations in progress;
• suspend autonomous power state management activities as described in section 8.1.17.2; and
• release stream identifiers for any open streams.
If a sanitize operation is in any of the following states (i.e., is in progress):
• Restricted Processing;
• Unrestricted Processing;
• Media Verification; or
• Post-Verification Deallocation,
then for each controller in the NVM subsystem:
• all I/O commands other than a Flush command shall be aborted with a status code of Sanitize In
Progress, unless otherwise specified;
• processing of a Flush command is specified in section 7.2;
