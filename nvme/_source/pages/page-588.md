---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 588
headings: ["8.1.22.8 Clearing a Reservation", "8.1.22.9 Reporting Reservation Status", "8.1.23 Rotational Media"]
---

# Source page 588

NVM Express® Base Specification, Revision 2.1

566
behavior of such a command is exactly the same as that described above with the RACQA field set to 001b
(Preempt), with two exceptions:
• After the atomic operation changes namespace reservation and registration state, all controllers
associated with any host whose reservation or registration is preempted by that atomic operation
are requested to abort all commands being processed that were addressed to the specified
namespace in the NSID field in the Reservation Acquire command  (refer to section 3.4.4 for the
definition of “being processed”); and
• Completion of the Reservation Acquire command shall not occur until all commands that are
requested to be aborted are completed, regardless of whether or not each command is actually
aborted.
As with the Abort command (refer to section 5.1), aborting a command  as a side effect of preempting a
reservation is best effort; as a command that is requested to be aborted may currently be at a point in
execution where that command is no longer able to be aborted or may have already completed, when a
Reservation Acquire or Abort Admin command is submitted. Although prompt execution of abort requests
reduces delay in completing the Reservation Acquire command, a command which is requested to be
aborted shall either be aborted or otherwise completed before the completion o f the Reservation Acquire
command.
When a registrant is unregistered as a result of actions described in this section, then a registration
preempted notification occurs on all controllers associated with a host that was unregistered other than the
host that issued the Reservation Acquire command.
When the type of reservation held on a namespace changes as a result of actions described in this section,
then a reservation released notification occurs on all controllers associated with hosts that remain
registrants of the namespace except the host that issued the Reservation Acquire command.
 Clearing a Reservation
A host that is a registrant may clear a reservation (i.e., force the release of a reservation held on the
namespace and unregister all registrants) by:
a) executing a Reservation Release command (refer to section 7.7);
b) setting the Reservation Release Action (RRELA) field to 001b (i.e., Clear); and
c) supplying the current reservation key associated with the host in the Current Reservation Key
(CRKEY) field.
If the value in the CRKEY field does not match the value used by the host to register with the namespace,
then the command shall be aborted with a status code of Reservation Conflict. If the host is not a registrant,
then the command is aborted with a status  code of Reservation Conflict. When a command to clear a
reservation is executed the following occur as an atomic operation: the reservation held on the namespace,
if any, is released, and all registrants are unregistered from the namespace.
A reservation preempted notification occurs on all controllers in the NVM subsystem that are associated
with hosts that have their registrations removed as a result of actions taken in this section except those
associated with the host that issued the Reservation Release command.
 Reporting Reservation Status
A host may determine the current reservation status associated with a namespace by executing a
Reservation Report command (refer to section 7.8).
 Rotational Media
Rotational media has different operational, endurance and performance characteristics than non-rotational
media (e.g., NAND). Rotational media utilizes electromechanical methods for accessing data.
Rotational media contains one or more spinning platters containing the media, and one or more actuators
that provide physical access to the data on that media (e.g., a hard disk drive or a CD -ROM).
A controller that supports namespaces that store user data on rotational media shall:
