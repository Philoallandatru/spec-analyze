---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 670
headings: ["8.3.4.4.3 Mapping of Common Authentication Messages to Authentication Commands", "8.3.4.5 DH-HMAC-CHAP Protocol", "8.3.4.5.1 Protocol Operations"]
---

# Source page 670

NVM Express® Base Specification, Revision 2.1

648
Figure 736: AUTH_Failure reason code explanations
Value Definition
01h Authentication failed: Authentication of the involved host or NVM subsystem failed.
02h Authentication protocol not usable:  The protocol descriptors proposed by the host do not
satisfy the security requirements of the controller (refer to section 8.3.4.4.1).
03h Secure channel concatenation mismatch: The SC_C value indicated by the host does not
satisfy the security requirements of the controller (refer to section 8.3.4.4.1).
04h Hash function not usable: The HashIDList proposed by the host does not satisfy the security
requirements of the controller (refer to section 8.3.4.5.2).
05h DH group not usable:  The DHgIDList proposed by the host does not satisfy the security
requirements of the controller (refer to section 8.3.4.5.2).
06h Incorrect payload: The payload of the received message is not correct.
07h Incorrect protocol message: The received message is not the expected next message in the
authentication protocol sequence.
All other values Reserved
8.3.4.4.3 Mapping of Common Authentication Messages to Authentication Commands
The AUTH_Negotiate message and the AUTH_Failure2 message are sent from the host to the controller,
therefore they are mapped to the Authentication Send command. The AUTH_Failure1 message is sent
from the controller to the host, therefore it is mapped to the Authentication Receive command.
 DH-HMAC-CHAP Protocol
8.3.4.5.1 Protocol Operations
DH-HMAC-CHAP is a key based Authentication and key management protocol that uses the Challenge
Handshake Authentication Protocol (CHAP, refer to RFC 1994) enhanced to use the Hashed Message
Authentication Code (HMAC) mechanism (refer to RFC 2104) with stronger hash functions and augmented
with an optional Diffie -Hellman (DH) exchange (refer to RFC 2631, clause 2.2.1). DH -HMAC-CHAP
provides bidirectional or unidirectional Authentication between a host and a controller.
The Diffie-Hellman part of the protocol is optional. When the Diffie-Hellman part of the protocol is not used,
DH-HMAC-CHAP is referred to as HMAC -CHAP. If insufficiently random keys are used (refer to section
8.3.4.5.7), HMAC-CHAP potentially allows a passive eavesdropper to discover the key through an off -line
dictionary attack, so its usage should be minimized. DH -HMAC-CHAP provides strong protection from
passive eavesdroppers. However, an active attacker could reduce the operation of this protocol so that only
HMAC-CHAP is used, and as a result gain sufficient information to mount an off -line dictionary attack on
the HMAC-CHAP key.
An implementation that supports DH -HMAC-CHAP authentication shall support DH -HMAC-CHAP with a
NULL DH exchange. All implementations of DH -HMAC-CHAP shall be configurable to require a DH
exchange (i.e., to not use HMAC-CHAP).
In order to authenticate with the DH -HMAC-CHAP protocol, each host and NVM subsystem shall be
provided with a DH -HMAC-CHAP key that is associated with the entity’s NQN. Two entities may
impersonate one another if they have the same key, therefore when the assigned keys are not different for
each entity there is a security vulnerability (refer to section 8.3.4.5.7).
To authenticate another entity, an entity is required to either:
a) know the key associated with the entity to be authenticated; or
b) rely on a third party that knows the key to verify the authentication (refer to section 8.3.4.5.11).
An example of a DH -HMAC-CHAP authentication transaction is shown in Figure 737, with the notation
shown in Figure 738. The DH -HMAC-CHAP_Success2 message that is shown as a dashed line is used
only for bidirectional authentication.
