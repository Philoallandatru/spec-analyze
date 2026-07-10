---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 586
headings: ["8.1.22.5 Acquiring a Reservation", "8.1.22.6 Releasing a Reservation", "8.1.22.7 Preempting a Reservation or Registration"]
---

# Source page 586

NVM Express® Base Specification, Revision 2.1

564
Successful completion of an unregister operation causes the host to no longer be a registrant of that
namespace. A host may unregister without regard to its current reservation key value by setting the IEKEY
bit to '1' in the Reservation Register command.
Unregistering by a host may cause a reservation held by the host to be released. If a host is the last
remaining reservation holder (i.e., the reservation type is Write Exclusive - All Registrants or Exclusive
Access - All Registrants) or is the only reser vation holder, then the reservation is released when the host
unregisters.
If a reservation is released and the type of the released reservation was Write Exclusive - Registrants Only
or Exclusive Access - Registrants Only, then a reservation released notification occurs on all controllers
associated with a registered host other than the host that issued the Reservation Register command.
 Acquiring a Reservation
In order for a host to obtain a reservation on a namespace, that host shall be a registrant of that namespace.
A registrant obtains a reservation by executing a Reservation Acquire command (refer to section 7.5),
clearing the Reservation Acquire Action (RACQA) field to 000b (Acquire), and supplying the current
reservation key associated with the host in the Current Reservation Key (CRKEY) field. The CRKEY value
shall match that used by the registrant to register with the namespace. If the CRKEY value does not match,
then the command is aborted with a status code of Reservation Conflict. If the host is not a registrant, then
the command is aborted with a status code of Reservation Conflict.
Only one reservation is allowed at a time on a namespace. If a registrant attempts to obtain a reservation
on a namespace that already has a reservation holder, then the command is aborted with  a status code of
Reservation Conflict. If a reservation holder attempts to obtain a reservation of a different type on a
namespace for which that host already is the reservation holder, then the command is aborted with a status
code of Reservation Conflict. If a reservation holder attempts to obtain a reservation of the same type on a
namespace for which that host already is the reservation holder , then it is not a Reservation Conflict and
the command is processed. A reservation holder may preempt a reservation to change the reservation type.
 Releasing a Reservation
Only a reservation holder is able to release a reservation held on a namespace. A host should release a
reservation using the following sequence:
a) executing a Reservation Release command (refer to section 7.7);
b) clearing the Reservation Release Action (RRELA) field to 000b (i.e., Release) ;
c) setting the Reservation Type (RTYPE) field to the type of reservation being released ; and
d) supplying the current reservation key associated with the host in the Current Reservation Key
(CRKEY) field. The CRKEY value shall match that used by the host to register with the namespace.
If the key value does not match, then the command is aborted with a status code of Reservation Conflict. If
the RTYPE field does not match the type of the current reservation, then the command completes with a
status code of Invalid Field in Command.
An attempt by a registrant to release a reservation using the Reservation Release command in the absence
of a reservation held on the namespace or when the host is not the reservation holder shall cause the
command to complete successfully, but shall have no effect on the controller or namespace.
When a reservation is released as a result of actions described in this section and the reservation type is
not Write Exclusive or Exclusive Access, a reservation released notification occurs on all controllers in the
NVM subsystem that are associated with hosts that are registrants except for controllers that are associated
with the host that issued the Reservation Release command.
 Preempting a Reservation or Registration
A host that is a registrant may preempt a reservation and/or registration by executing a Reservation Acquire
command (refer to section 7.5), specifying:
