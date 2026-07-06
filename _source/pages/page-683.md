---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 683
headings: []
---

# Source page 683

NVM Express® Base Specification, Revision 2.1

661
Figure 750: DH-HMAC-CHAP with TLS Concatenation Configurations
Configuration Description
Authentication Required
TLS Disabled Authentication of a remote entity required, set up of a secure
channel not allowed
TLS Permitted Authentication of a remote entity required, set up of a secure
channel allowed
TLS Required Authentication and set up of a secure channel with a remote entity
required
NVM subsystems that support secure channel concatenation with the TLS protocol shall set the ATR and
ASCR bits in the AUTHREQ field in the response of a successful Connect command on an Admin Queue
over a TCP channel without TLS (refer to Figure 548) as defined in Figure 751.
Figure 751: NVM Subsystem AUTHREQ Settings for DH-HMAC-CHAP with
TLS Concatenation
Configuration ASCR ATR
Authentication Disabled TLS Disabled 0 0
Authentication Permitted TLS Disabled 0 0
TLS Permitted 0 0
Authentication Required
TLS Disabled 0 1
TLS Permitted 0 1
TLS Required 1 0
Hosts that support secure channel concatenation with the TLS protocol shall behave as defined Figure 752,
according to the ATR and ASCR bits in the AUTHREQ field received in the response of a successful
Connect command on an Admin Queue over a TCP channel without TLS (refer to Figure 548).
Figure 752: DH-HMAC-CHAP with TLS Concatenation Host Behavior
Host Configuration ASCR ATR Action
Authentication
Disabled TLS Disabled
0 0 Do not begin an authentication transaction
0 1 Disconnect from the NVM subsystem 1 any
Authentication
Permitted TLS Disabled
0 0
If the TASC field in the Discovery Log Page Entry for
the remote entity is set to 01b or set to 10b, then begin
an authentication transaction with the SC_C field set to
NOSC.
If the TASC field in the Discovery Log Page Entry for
the remote entity is cleared to 00b or set to 11b or if no
Discovery Log Page Entry for the remote entity has
been retrieved, then do not begin an authentication
transaction.
0 1 Begin an authentication transaction with the SC_C field
set to NOSC
1 any Disconnect from the NVM subsystem
