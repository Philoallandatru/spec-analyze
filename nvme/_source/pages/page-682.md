---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 682
headings: []
---

# Source page 682

NVM Express® Base Specification, Revision 2.1

660
Figure 748: DH-HMAC-CHAP Host Behavior
Host Configuration ASCR ATR Action
Authentication Disabled
0 0 Do not begin an authentication transaction
0 1 Disconnect from the NVM subsystem 1 any
Authentication Permitted
0 0
If the TASC field in the Discovery Log Page Entry for the
remote entity is set to 01b or set to 10b, then begin an
authentication transaction with the SC_C field set to NOSC.
If the TASC field in the Discovery Log Page Entry for the
remote entity is cleared to 00b or set to 11b or if no Discovery
Log Page Entry for the remote entity has been retrieved, then
do not begin an authentication transaction.
0 1 Begin an authentication transaction with the SC_C field set to
NOSC
1 any Disconnect from the NVM subsystem
Authentication Required 0 any Begin an authentication transaction with the SC_C field set to
NOSC
1 any Disconnect from the NVM subsystem
NVM subsystems that do not support secure channel concatenation with the TLS protocol shall behave as
defined in Figure 749, according to the SC_C field in a received AUTH_Negotiate message (refer to section
8.3.4.4.1).
Figure 749: DH-HMAC-CHAP NVM Subsystem Behavior
Subsystem
Configuration SC_C Value Action
Authentication Disabled Any Disconnect from the host
Authentication Permitted
NOSC Participate in the authentication transaction
NEWTLSPSK or
REPLACETLSPSK Disconnect from the host
Authentication Required
NOSC Participate in the authentication transaction
NEWTLSPSK or
REPLACETLSPSK Disconnect from the host
For example, consider a host that supports DH -HMAC-CHAP without secure channel concatenation with
the TLS protocol configured with Authentication Permitted and an NVM subsystem that supports DH -
HMAC-CHAP without secure channel concatenation with the TLS pr otocol configured with Authentication
Required. As defined in Figure 747, the NVM subsystem clears the ASCR bit to ‘0’ and sets the ATR bit to
‘1’ in the Connect response. As defined in Figure 748, upon receiving the Connect response the host begins
an authentication transaction with the SC_C field set to NOSC. As defined in Figure 749, the NVM
subsystem participates in the authentication transaction.
DH-HMAC-CHAP implementations that support secure channel concatenation with the TLS protocol shall
support configuring use of DH-HMAC-CHAP with secure channel concatenation with the TLS protocol using
one or more of the configurations shown in Figure 750.
Figure 750: DH-HMAC-CHAP with TLS Concatenation Configurations
Configuration Description
Authentication Disabled TLS Disabled Authentication and set up of a secure channel with a remote entity
not allowed
Authentication Permitted
TLS Disabled Authentication of a remote entity allowed, set up of a secure
channel not allowed
TLS Permitted Authentication and set up of a secure channel with a remote entity
allowed
