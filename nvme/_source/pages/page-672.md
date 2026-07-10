---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 672
headings: ["8.3.4.5.2 DH-HMAC-CHAP Authentication Protocol Descriptor"]
---

# Source page 672

NVM Express® Base Specification, Revision 2.1

650
1) The authentication transaction shall begin with the host sending the common AUTH_Negotiate
message to negotiate the authentication protocol to use and its associated parameters (refer to
section 8.3.4.4.1). The AUTH_Negotiate message carries the transaction identifier (T_ID) for the
entire authentication transaction and the list of authentication protocol descriptors for the
authentication protocols that may be used in this authentication transaction. For DH-HMAC-CHAP,
the authentication protocol descriptor includes the list of hash functions (HashIDList) and Diffie -
Hellman group identifiers (DHgIDList) that may be used in this authentication protocol transaction.
2) If the parameters of the received DH -HMAC-CHAP protocol descriptor are compatible with the
controller’s policies, then the controller shall reply with a DH -HMAC-CHAP_Challenge message
(refer to section 8.3.4.5.3) carrying the same transaction identifier value (T_ID) received in the
AUTH_Negotiate message, the identifiers of the hash function (HashID) and the DH group (DHgID)
selected for use among the ones proposed by the host in the AUTH_Negotiate message, a
sequence number (S 1), a random challenge value (C 1), and the DH exponential (g x mod p). If the
controller selects a NULL DH group identifier, then the DH portion of the DH-HMAC-CHAP protocol
shall not be used, and the protocol reduces to a HMAC-CHAP transaction.
3) If the received DH -HMAC-CHAP_Challenge message is valid, then the host shall send a DH -
HMAC-CHAP_Reply message (refer to section 8.3.4.5.11) carrying the same transaction identifier
value (T_ID), the response R 1 to the challenge value C 1, and its own DH exponential (g y mod p).
The DH Value Length shall be cleared to 0h if the controller has sent a NULL DH group identifier
in the DH-HMAC-CHAP_Challenge message. If bidirectional authentication is requested, then the
DH-HMAC-CHAP_Reply message shall carry also a sequence number S2 and a random challenge
value C 2 that differs from the challenge value C 1 received in the DH -HMAC-CHAP_Challenge
message.
4) If the authentication verification by the controller succeeds, then the controller shall reply with a
DH-HMAC-CHAP_Success1 message (refer to section 8.3.4.5.5) carrying the same transaction
identifier value (T_ID). If bidirectional authentication was requested, then the DH -HMAC-
CHAP_Success1 message shall also carry the response R 2 to the challenge value C 2. If the
authentication verification fails, then the controller shall send an AUTH_Failure1 message and
disconnect the NVMe over Fabrics connection upon transmitting it.
5) The authentication transaction ends here, unless bidirectional authentication has been requested.
In this case, as shown by the dashed arrow in Figure 737, if the authentication verification by the
host succeeds, then the host shall send a DH -HMAC-CHAP_Success2 message (refer to section
8.3.4.5.6) carrying the same transaction identifier value (T_ID). If the authentication verification
fails, then the host shall send an AUTH_Failure2 message and disconnect the NVMe over Fabrics
connection upon transmitting it.
If the controller receives a message that is not the expected next message in the DH-HMAC-CHAP protocol
sequence, then the controller shall:
• reply with an AUTH_Failure1 message having reason code ‘Authentication failure’ and reason code
explanation ‘Incorrect protocol message’; and
• disconnect the NVMe over Fabrics connection upon transmitting the AUTH_Failure1 message.
If the host receives a message that is not the expected next message in the DH -HMAC-CHAP protocol
sequence, then the host shall:
• reply with an AUTH_Failure2 message having reason code ‘Authentication failure’ and reason code
explanation ‘Incorrect protocol message’; and
• disconnect the NVMe over Fabrics connection upon transmitting the AUTH_Failure2 message.
The payload format of a message shall be validated before performing any other security computation.
8.3.4.5.2 DH-HMAC-CHAP Authentication Protocol Descriptor
The authentication protocol descriptor for DH -HMAC-CHAP (refer to section 8.3.4.4.1) is shown in Figure
739.
