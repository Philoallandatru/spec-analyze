---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 681
headings: ["8.3.4.5.10 Mapping of DH-HMAC-CHAP Messages to Authentication Commands", "8.3.4.5.11 DH-HMAC-CHAP Configuration"]
---

# Source page 681

NVM Express® Base Specification, Revision 2.1

659
This generated PSK should be replaced periodically (e.g., every hour) or on demand by performing a
reauthentication on the Admin queue of the association with the appropriate SC_C value (refer to section
8.3.4.1).
The host may request secure channel concatenation with the TLS protocol by setting the SC_C field in the
AUTH_Negotiate message to NEWTLSPSK while performing only unidirectional authentication. In this case
the host shall still send a challenge value C 2 to the controller and clear the sequence number S 2 to 0h to
indicate that controller authentication is not requested.
8.3.4.5.10 Mapping of DH-HMAC-CHAP Messages to Authentication Commands
The DH-HMAC-CHAP_Reply message and the DH -HMAC-CHAP_Success2 message are sent from the
host to the controller, therefore they are mapped to the Authentication Send command. The DH -HMAC-
CHAP_Challenge message and the DH -HMAC-CHAP_Success1 message are sent f rom the controller to
the host, therefore they are mapped to the Authentication Receive command.
8.3.4.5.11 DH-HMAC-CHAP Configuration
DH-HMAC-CHAP implementations that do not use an AVE (refer to section  8.3.4.6 for AVE information)
shall be able to be provisioned with their own DH -HMAC-CHAP secret and with a verification secret per
each remote entity that is able to be authenticated. DH -HMAC-CHAP implementations that use an AVE
shall be able to be provisioned with their own DH -HMAC-CHAP secret and the parameters for accessing
the AVE (refer to section 8.3.4.6.2).
This section uses the term configuration to indicate an NVMe internal state that controls the use of DH -
HMAC-CHAP. That internal state may or may not be externally configurable for a host or NVM subsystem.
For an NVMe/TCP implementation that supports DH-HMAC-CHAP, a DH-HMAC-CHAP configuration may
apply to:
• a single connection;
• a group of connections; or
• all connections.
DH-HMAC-CHAP implementations that do not support secure channel concatenation with the TLS protocol
shall support configuring use of DH-HMAC-CHAP using one or more of the configurations shown in Figure
746.
Figure 746: DH-HMAC-CHAP Configurations
Configuration Description
Authentication Disabled Authentication of a remote entity not allowed
Authentication Permitted Authentication of a remote entity allowed
Authentication Required Authentication of a remote entity required
NVM subsystems that do not support secure channel concatenation with the TLS protocol shall set the ATR
and ASCR bits in the AUTHREQ field in the response of a successful Connect command (refer to Figure
548) as defined in Figure 747.
Figure 747: NVM Subsystem AUTHREQ Settings for DH-
HMAC-CHAP
Configuration ASCR ATR
Authentication Disabled 0 0
Authentication Permitted 0 0
Authentication Required 0 1
Hosts that do not support secure channel concatenation with the TLS protocol shall behave as defined in
Figure 748, according to the ATR and ASCR bits in the AUTHREQ field received in the response of a
successful Connect command (refer to Figure 548).
