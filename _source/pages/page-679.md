---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 679
headings: ["8.3.4.5.6 DH-HMAC-CHAP_Success2 Message", "8.3.4.5.7 DH-HMAC-CHAP Security Requirements"]
---

# Source page 679

NVM Express® Base Specification, Revision 2.1

657
8.3.4.5.6 DH-HMAC-CHAP_Success2 Message
The DH-HMAC-CHAP_Success2 message is sent from the host to the controller and indicates that the host
has successfully authenticated the controller. The format of the DH -HMAC-CHAP_Success2 message is
shown in Figure 745.
Figure 745: DH-HMAC-CHAP_Success2 message format
Bytes Description
0 Authentication Type (AUTH_TYPE): 01h (i.e., DH-HMAC-CHAP)
1 Authentication Identifier (AUTH_ID): 04h (i.e., DH-HMAC-CHAP_Success2)
3:2 Reserved
5:4 Transaction Identifier (T_ID): 16-bit transaction identifier
15:6 Reserved
8.3.4.5.7 DH-HMAC-CHAP Security Requirements
In order to authenticate with the DH -HMAC-CHAP protocol, each host or controller uses a DH -HMAC-
CHAP key that is associated with the entity’s NQN. A DH-HMAC-CHAP key is unidirectional (i.e., used only
for one direction of an authentication transaction). A DH -HMAC-CHAP key should not be associated with
more than one NQN as this opens security vulnerabilities. All DH -HMAC-CHAP implementations should
check for use of the same key with more than one NQN and should generate an administrative warning if
this situation occurs (e.g., as a result of configuring a DH -HMAC-CHAP key to verify authentication of
another entity).
The DH-HMAC-CHAP key is derived from an administratively configured secret (refer to section 8.3.4.5.8).
Each host and NVM subsystem shall support:
• transforming the provided secret into a key applying the HMAC function using the hash function
specified in the secret representation (refer to section 8.3.4.5.8) with the secret as HMAC key to
the concatenation of its own NQN not including the null terminator and the seventeen ASCII
characters “NVMe -over-Fabrics” (i.e., key = HMAC(secret, NQN || ”NVMe -over-Fabrics”)). This
transformation ensures the resulting key  is uniquely associated with the entity identified by the
NQN; and
• using the provided secret as a key. This is intended for use with key management solutions able to
ensure that key is uniquely associated with the entity identified by the NQN.
NVM subsystems should support the ability to use a different NVM subsystem key with each host. Hosts
should support the ability to use a different host key with each NVM subsystem. NVM subsystems should
support the ability to use a different NVM subsystem secret with each host. Hosts should support the ability
to use a different host secret with each NVM subsystem.
If an implementation of NVMe over Fabrics is capable of functioning as both a host and an NVM subsystem,
then that implementation shall use either:
• one NQN for the host functionality and a different NQN for the NVM subsystem functionality; or
• one NQN for both host functionality and NVM subsystem functionality.
DH-HMAC-CHAP implementations may reuse a DH exponential (e.g., g x mod p or gy mod p). The primary
risk in allowing reuse of a DH exponential is replay of a prior authentication sequence based on the attacker
reusing the other exponential. For DH-HMAC-CHAP, replay is prevented with extremely high probability by
the requirement that all challenges be randomly generated. See section 2.12 of RFC 7296 for guidance on
DH exponential reuse.
The security of the DH-HMAC-CHAP protocol requires secrets, challenges, and DH exponents (i.e., x and
y) to be generated from actual randomness. For a discussion of randomness and sources of randomness,
refer to RFC 4086.
Implementations shall use a cryptographic random number generator that should be seeded with at least
256 bits of entropy to generate random numbers for this protocol. The secret provisioning mechanism for
