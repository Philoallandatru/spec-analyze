---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 704
headings: ["B.6.3. Retried Command Does Not Succeed"]
---

# Source page 704

NVM Express® Base Specification, Revision 2.1

682
Figure 760: Non-Idempotent Command

B.6.3. Retried Command Does Not Succeed
In the example shown in Figure 761, the host loses communication with Controller 1 and does not receive
a response to a Reservation Register command that unregisters the host (refer to section 7.6). The host
ensures that no further controller processing of that command is possible (refer to section 9.6.2), and then
retries that command on Controller 2. As a result of the original command unregistering the host, the host
is no longer a registrant, and for that reason , the controller returns a status code of Reservation Conflict
(refer to section 8.1.22.4).
Figure 761: Retried Command Does Not Succeed

This example shows why an error status code is able to be returned if a non-idempotent command is retried
after the original command has been processed. An analogous example is possible for the Compare and
Write fused operation (refer to the Fused Operatio n section of the NVM Command Set Specification)
because that fused operation is not idempotent
Host detects a loss
of communication.
Host


Namespace Management create
namespace
Controller 1
 Controller 2
Retry: Namespace Management
create namespace

Two Namespaces are created!
Command takes effect.
Success
Controller processing command.
Host determines that Controller 1
has stopped processing the
command.
Host detects a loss
of communication.
Host


Reservation Register release registration
Controller 1
 Controller 2
Retry: Reservation Register release
registration

Command does not complete successfully!
Command takes effect.
Reservation Conflict
Controller processing command
Host determines that Controller 1 has
stopped processing the command.
