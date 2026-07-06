---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 702
headings: ["B.5 Updating Controller Doorbell Properties using a Shadow Doorbell Buffer", "B.5.1. Shadow Doorbell Buffer Overview", "B.5.2. Example Algorithm for Controller Doorbell Property Updates", "B.6 Examples of Incorrect Command Retry Handling After Communication Loss", "B.6.1. Write after Write"]
---

# Source page 702

NVM Express® Base Specification, Revision 2.1

680
B.5 Updating Controller Doorbell Properties using a Shadow Doorbell Buffer
B.5.1. Shadow Doorbell Buffer Overview
Controllers that support the Doorbell Buffer Config command are typically emulated controllers where this
feature is used to enhance the performance of host software running in Virtual Machines. If supported by
the controller, host software may enable Shadow Doorbell buffers by submitting the Doorbell Buffer Config
command (refer to section 5.2.5).
After the completion of the Doorbell Buffer Config command, host software shall submit commands by
updating the appropriate entry in the Shadow Doorbell buffer instead of updating the controller's
corresponding doorbell property. If updating an entry in the Shadow Doorbell buffer changes the value from
being less than or equal to the value of the corresponding EventIdx buffer entry to being greater than that
value, then the host shall also update the controller's corresponding do orbell property to match the value
of that entry in the Shadow Doorbell buffer. Queue wrap conditions shall be taken into account in all
comparisons in this paragraph.
The controller may read from the Shadow Doorbell buffer and update the EventIdx buffer at any time (e.g.,
before the host writes to the controller's doorbell property).
B.5.2. Example Algorithm for Controller Doorbell Property Updates
Host software may use modular arithmetic where the modulus is the queue depth to decide if the controller
doorbell property should be updated, specifically:
• Compute X as the new doorbell value minus the corresponding EventIdx value, modulo queue
depth; and
• Compute Y as the new doorbell value minus the old doorbell value in the shadow doorbell buffer,
also modulo queue depth.
If X is less than or equal to Y, the controller doorbell property should be updated with the new doorbell
value.
B.6 Examples of Incorrect Command Retry Handling After Communication Loss
Section 9.6.3 describes requirements for host retry of outstanding commands after communication loss. In
this situation, the response of a command is unknown and hence the host has no information about the
extent, if any, to which the controller has processed that comm and. Many commands are not safe to
unconditionally retry if they have been processed in part or completely. This annex describes examples of
problematic situations caused by retrying an outstanding command without regard to the consequences of
that retry.
B.6.1. Write after Write
In the example shown in Figure 759, the host loses communication with Controller 1 and does not receive
a response from Controller 1 for an idempotent command that changes user data at location X to A (e.g.,
an NVM Command Set Write command). The following events occur:
• The host retries that command on Controller 2 (Retry: Write A at Location X), and the retry succeeds
quickly.
• The completion of that retry leads to the host subsequently submitting a command that changes
the user data at the same location to B (Write B at Location X).
• During this time, Controller 1 has been processing the original outstanding command (Write A at
Location X), and that command’s change of user data at location X to A finally takes effect after the
user data at location X has already been changed to B.
The final outcome is that the user data at location X is A, which is incorrect and an example of data
corruption.
For an idempotent command that changes user data or NVM subsystem state, this example shows why
the host should not report the results of that command, including any retry of that command, to higher-level
