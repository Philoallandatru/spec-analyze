---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 686
headings: []
---

# Source page 686

NVM Express® Base Specification, Revision 2.1

664
Figure 754: Example of DH-HMAC-CHAP authentication transaction with AVE
ControllerHost AVE
: NVMe protocol
: AVE access protocol
DH-HMAC-CHAP_Access-Request
(T_ID, SC_C, HashID,  H   S1, Ca1, R1, NQNh)
DH-HMAC-CHAP_Access-Result (AuthRes)
DH-HMAC-CHAP_Access-Request
(T_ID, SC_C, HashID,  C   S2, Ca2, R2, NQNc)
DH-HMAC-CHAP_Access-Result (AuthRes)
[DH-HMAC-CHAP_Success2]
(T_ID)
DH-HMAC-CHAP_Success1
(T_ID, [R2])
DH-HMAC-CHAP_Reply
(T_ID, R1, gy mod p, [S2, C2])
DH-HMAC-CHAP_Challenge
(T_ID, HashID, DHgID, S1, C1, gx mod p)
AUTH_Negotiate
(T_ID, SC_C, AuthID, HashIDList, DHgIDList)

As shown in Figure 754, a controller using the AVE service delegates to the AVE the verification of the
response R1 received from the host by passing the relevant DH -HMAC-CHAP authentication transaction
parameters to the AVE through a DH -HMAC-CHAP_Access-Request message. A host using the AVE
service delegates to the AVE the verification of the response R2 received from the controller by passing the
relevant DH -HMAC-CHAP authentication transaction parameters to the AVE through a DH -HMAC-
CHAP_Access-Request message. In both cases, the AVE replies with a DH-HMAC-CHAP_Access-Result
message containing the result of the authentication verification (refer to section 8.3.4.6.3).
Use of the AVE service by an NVMe entity is optional and is determined by configuration of the NVMe
entity. If an NVMe entity uses the AVE, then provisioning of DH -HMAC-CHAP information on that entity is
reduced to only that entity’s DH-HMAC-CHAP secret (refer to section 8.3.4.5.8) and the parameters (refer
to section 8.3.4.6.2) for accessing the AVE (i.e., no DH -HMAC-CHAP keys are required to be provisioned
for verification of responses received from other NVMe entities).
An AVE is required to maintain the following information for each NVMe entity:
• The NQN of that entity (i.e., NQNe),
• the DH-HMAC-CHAP key associated with that entity (i.e., Ke), and
• the PSK shared between that entity and the AVE (i.e., PSK ea).
Ke is used to perform the authentication verification function (refer to section 8.3.4.6.3) and PSKea is used
to establish a secure connection with the AVE (refer to section 8.3.4.6.2). An AVE shall support all hash
functions defined for DH-HMAC-CHAP (refer to section 8.3.4.5.2).
To facilitate dynamic discovery of the transport addresses of an AVE through a Discovery Controller (refer
to section 8.3.4.6.4) and to simplify establishing a secure connection to an AVE (refer to section 8.3.4.6.2),
an AVE is identified at by an NQN (NQNAVE).
