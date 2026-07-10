---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 671
headings: []
---

# Source page 671

NVM Express® Base Specification, Revision 2.1

649
Figure 737: Example of DH-HMAC-CHAP authentication transaction
AUTH_Negotiate
(T_ID, SC_C, AuthID, HashIDList, DHgIDList)
DH-HMAC-CHAP_Reply
(T_ID, R1, gy mod p, [S2, C2])
DH-HMAC-CHAP_Challenge
(T_ID, HashID, DHgID, S1, C1, gx mod p)
DH-HMAC-CHAP_Success1
(T_ID, [R2])
[DH-HMAC-CHAP_Success2]
(T_ID)
Host Controller

Figure 738: Mathematical notations for DH-HMAC-CHAP
Symbols Description
NQNc, NQNh NQN of the NVM subsystem that contains the controller and NQN of the host
Kc, Kh DH-HMAC-CHAP key of the NVM subsystem that contains the controller and
DH-HMAC-CHAP key of the host
p, g Modulus (p) and generator (g) of the chosen DH group (refer to Figure 741)
x, y Random numbers used as exponents in a DH exchange
C1, C2 Random challenge values
Ca1, Ca2 Augmented challenge values
S1, S2 32-bit sequence numbers
R1, R2 Reply values
T_ID Authentication transaction identifier
SC_C Secure channel concatenation indication
H( ) One-way hash function (refer to Figure 740)
HMAC(K, Str) HMAC function (refer to RFC 2104) with key K on string Str using hash function H( )
|| Concatenation operation
KS Session key
When used with a non -NULL DH exchange, the DH -HMAC-CHAP protocol is able to generate a session
key KS to be used to establish a TLS session between host and controller (refer to section 8.3.4.5.9).
For an NVM subsystem, the controller is the entity running the protocol, using the identity and credentials
of the NVM subsystem. The DH-HMAC-CHAP protocol proceeds in the following order:
