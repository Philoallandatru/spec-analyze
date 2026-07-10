---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 685
headings: ["8.3.4.6 DH-HMAC-CHAP Authentication Verification Entity", "8.3.4.6.1 Overview"]
---

# Source page 685

NVM Express® Base Specification, Revision 2.1

663
Figure 753: DH-HMAC-CHAP with TLS Concatenation NVM Subsystem Behavior
Subsystem Configuration SC_C Value Action
Authentication
Required
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
TLS Required
NOSC or
REPLACETLSPSK Disconnect from the host
NEWTLSPSK
Participate in the authentication transaction and
establish a TLS channel as described in section
8.3.4.3
As an example, consider a host that supports DH -HMAC-CHAP with secure channel concatenation with
the TLS protocol configured with Authentication Permitted, TLS Permitted and an NVM subsystem that
supports DH -HMAC-CHAP with secure channel concatenation with  the TLS protocol configured with
Authentication Required, TLS Required. As defined in Figure 751, the NVM subsystem sets the ASCR bit
to ‘1’ and clears the ATR bit to ‘0’ in the Connect response. As defined in Figure 752, upon receiving the
Connect response the host begins an authentication transaction with the SC_C field set to NEWTLSPSK.
As defined in Figure 753, the NVM subsystem participates in the authentication transaction and establishes
a TLS channel.
 DH-HMAC-CHAP Authentication Verification Entity
8.3.4.6.1 Overview
A DH-HMAC-CHAP Authentication Verification Entity (AVE) is a service that performs the DH-HMAC-CHAP
authentication verification function on behalf of an NVMe entity (i.e., a host or a controller). An example of
a DH-HMAC-CHAP authentication transaction with AVE is shown in Figure 754, using the notation shown
in Figure 738.
