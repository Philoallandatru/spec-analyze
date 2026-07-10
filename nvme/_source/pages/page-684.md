---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 684
headings: []
---

# Source page 684

NVM Express® Base Specification, Revision 2.1

662
Figure 752: DH-HMAC-CHAP with TLS Concatenation Host Behavior
Host Configuration ASCR ATR Action
TLS Permitted
0 0
If the TASC field in the Discovery Log Page Entry for
the remote entity is set to 01b, then begin an
authentication transaction with the SC_C field set to
NOSC.
If the TASC field in the Discovery Log Page Entry for
the remote entity is set to 10b, then begin an
authentication transaction with the SC_C field set to
NEWTLSPSK.
If the TASC field in the Discovery Log Page Entry for
the remote entity is cleared to 00b or set to 11b or if no
Discovery Log Page Entry has been retrieved for the
remote entity, then do not begin an authentication
transaction.
0 1 Begin an authentication transaction with the SC_C field
set to NOSC
1 any Begin an authentication transaction with the SC_C field
set to NEWTLSPSK
Authentication
Required
TLS Disabled 0 any Begin an authentication transaction with the SC_C field
set to NOSC
1 any Disconnect from the NVM subsystem
TLS Permitted
0 any Begin an authentication transaction with the SC_C field
set to NOSC
1 any Begin an authentication transaction with the SC_C field
set to NEWTLSPSK
TLS Required any any Begin an authentication transaction with the SC_C field
set to NEWTLSPSK
NVM subsystems that support secure channel concatenation with the TLS protocol shall behave as defined
in Figure 753, according to the SC_C field in a received AUTH_Negotiate message on an Admin Queue
over a TCP channel without TLS (refer to section 8.3.4.4.1).
Figure 753: DH-HMAC-CHAP with TLS Concatenation NVM Subsystem Behavior
Subsystem Configuration SC_C Value Action
Authentication
Disabled TLS Disabled any Disconnect from the host
Authentication
Permitted
TLS Disabled
NOSC Participate in the authentication transaction
NEWTLSPSK or
REPLACETLSPSK Disconnect from the host
TLS Permitted
NOSC Participate in the authentication transaction
NEWTLSPSK
Participate in the authentication transaction and
establish a TLS channel as described in section
8.3.4.3
REPLACETLSPSK Disconnect from the host
