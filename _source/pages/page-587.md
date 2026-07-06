---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 587
headings: []
---

# Source page 587

NVM Express® Base Specification, Revision 2.1

565
• the Reservation Acquire Action (RACQA) field  set to 001b ( i.e., Preempt) or set to 010b ( i.e.,
Preempt and Abort); and
• the current reservation key associated with the host in the Current Reservation Key (CRKEY) field.
The CRKEY value shall match that used by the registrant to register with the namespace. If the CRKEY
value does not match, then the command is aborted with a status code of Reservation Conflict. The preempt
actions that occur are dependent on the type of reservation held on the namespace, if any, and the value
of the Preempt Reservation Key (PRKEY) field in the command. If the host is not a registrant, then the
command is aborted with a status code of Reservation Conflict. The remainder of this section assumes that
the host is a registrant.
If the existing reservation type is not Write Exclusive - All Registrants and not Exclusive Access - All
Registrants, then the actions performed by the command depend on the value of the PRKEY field as
follows:
a) If the PRKEY field value matches the reservation key of the current reservation holder, then the
following occur as an atomic operation:
• all registrants with a matching reservation key other than the host that issued the command
are unregistered;
• the reservation is released; and
• a new reservation is created of the type specified by the Reservation Type (RTYPE) field in the
command for the host that issued the command as the reservation key holder;
or
b) If the PRKEY field value does not match that of the current reservation holder and is not equal to
0h, then registrants whose reservation key matches the value of the PRKEY field are unregistered.
If the PRKEY field value does not match that of the current  reservation holder and is equal to 0h,
then the command is aborted with a status code of Invalid Field in Command.
If the existing reservation type is Write Exclusive - All Registrants or Exclusive Access - All Registrants,
then the actions performed by the command depend on the value of the PRKEY field as follows:
a) If the PRKEY field value is 0h, then the following occurs as an atomic operation:
• all registrants other than the host that issued the command are unregistered;
• the reservation is released; and
• a new reservation is created of the type specified by the Reservation Type (RTYPE) field in the
command for the host that issued the command as the reservation key holder;
or
b) If the PRKEY value is non -zero, then registrants whose reservation key matches the value of the
PRKEY field are unregistered. If the PRKEY value is non-zero and there are no registrants whose
reservation key matches the value of the PRKEY field, the controller shall abort the command with
a status code of Reservation Conflict.
If there is no reservation held on the namespace, then execution of the command causes registrants whose
reservation key match the value of the PRKEY field to be unregistered.
If the existing reservation type is not Write Exclusive - All Registrants and not Exclusive Access - All
Registrants, then a reservation holder may preempt itself using the above mechanism. When a host
preempts itself, the following occurs as an atomic operation:
• registration of the host is maintained;
• the reservation is released; and
• a new reservation is created for the host of the type specified by the RTYPE field.
A host may abort commands as a side effect of preempting a reservation by executing a Reservation
Acquire command (refer to section 7.5) and setting the RACQA field to 010b (Preempt and Abort). The
