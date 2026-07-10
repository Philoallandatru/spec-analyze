---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 585
headings: ["8.1.22.3 Registering", "8.1.22.4 Unregistering"]
---

# Source page 585

NVM Express® Base Specification, Revision 2.1

563
 Registering
Prior to establishing a reservation on a namespace, a host shall become a registrant of that namespace by
registering a reservation key. This reservation key may be used by the host as a means of identifying the
registrant (host), authenticating the regist rant, and preempting a failed or uncooperative registrant. The
value of the reservation key used by a host and the method used to select its value is outside the scope of
this specification.
Registering a reservation key with a namespace creates an association between a host and a namespace.
A host that is a registrant of a namespace may use any controller with which that host is associated (i.e.,
that has the same Host Identifier, refer to se ction 5.1.25.1.28) to access that namespace as a registrant.
Thus, a host is only required to register on a single controller to become a registrant of the namespace on
all controllers in the NVM subsystem that have access to the namespace and are associated with the host.
If a host attempts to access a dispersed namespace that is able to be accessed by controllers in multiple
participating NVM subsystems, then that host is only required to register on a single controller in a single
participating NVM subsystem to become a registrant of the dispersed namespace on all controllers in all
participating NVM subsystems that have access to the dispersed namespace and are associated with the
host.
A host registers a reservation key by executing a Reservation Register command (refer to section 7.6) on
the namespace with the Reservation Register Action (RREGA) field cleared to 000b (i.e., Register
Reservation Key) and supplying a reservation key in the New Reservation Key (NRKEY) field.
A host that is a registrant of a namespace may register the same reservation key value multiple times with
the namespace on the same or different controllers. For a Reservation Register command with the RREGA
field cleared to 000b:
a) the IEKEY field shall be ignored; and
b) if a host that is already a registrant of a namespace attempts to register with that namespace using
a different reservation key value, then the command shall be aborted with  a status code of
Reservation Conflict.
There are no restrictions on the reservation key value used by hosts with different Host Identifiers. For
example, multiple hosts may all register with the same reservation key value.
A host that is a registrant of a namespace may replace the existing reservation key value for that
namespace by executing a Reservation Register command on the namespace with the:
a) RREGA field set to 010b (i.e., Replace Reservation Key);
b) current reservation key in the Current Reservation Key (CRKEY) field; and
c) new reservation key in the NRKEY field.
The current reservation key value shall be replaced by the new reservation key value in all controllers to
which the namespace is attached that have the same Host Identifier as the Host Identifier of the controller
processing the command. For dispersed namespaces, this requirement includes all participating NVM
subsystems. If the contents of the CRKEY field do not match the key currently associated with the host,
then the command shall be aborted with a status code of Reservation Conflict. A host may replac e its
reservation key without regard to its registration status or current reservation key value by setting the Ignore
Existing Key (IEKEY) bit to '1' in the Reservation Register command. Replacing a reservation key has no
effect on any reservation that may be held on the namespace.
 Unregistering
A host that is a registrant of a namespace may unregister with the namespace by executing a Reservation
Register command (refer to section 7.6) on the namespace with the RREGA field set to 001b (i.e.,
Unregister Reservation Key) and supplying its current reservation key in the CRKEY field. If the contents
of the CRKEY field do not match the key currently associated with the host, then the comman d is aborted
with a status code of Reservation Conflict. If the host is not a registrant, then the command is aborted with
a status code of Reservation Conflict.
