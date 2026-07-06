---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 688
headings: []
---

# Source page 688

NVM Express® Base Specification, Revision 2.1

666
Figure 755: DH-HMAC-CHAP_Access-Request PDU format
Bytes Description
03 PDU Data Offset (PDO): Reserved
07:04 PDU Length (PLEN): total length of the PDU in bytes
15:08 Identifier (ID): 64-bit identifier used to match Access-Request and Access-Result
PDUs
16 Hash Length (HL): Length in bytes of the selected hash function
17 Hash Identifier (HashID): Identifier of selected hash function
19:18 Transaction Identifier (T_ID): 16-bit authentication transaction identifier
20 Secure Channel Concatenation  (SC_C): Secure Channel concatenation
indication
21 Responder’s Role (RESPR): ‘H’ if host, ‘C’ if controller
22 NQN Responder Length (NQNRlen): Length of the responder’s NQN
23 Reserved
27:24 Sequence Number (SEQN): Sequence number S
27+HL:28 Augmented Challenge Value (ACV): Challenge Ca
27+2*HL:28+HL Response Value (RESPV): Response R
NQNRlen+27+2*HL:28+2*HL NQN of Responder (NQNR): Responder’s NQN
The DH-HMAC-CHAP_Access-Request PDU contains the parameters exchanged by the host and the
controller during a DH-HMAC-CHAP authentication transaction. The responder is the entity that replied to
a DH-HMAC-CHAP challenge sent by an authenticator.
Referring to Figure 754, when the controller transmits the DH -HMAC-CHAP_Access-Request PDU, the
parameters are instantiated as follows:
• Responder’s Role: ‘H’
• Sequence Number: S1
• Augmented Challenge Value: Ca1
• Response Value: R1
• NQNR: NQNh
• HashID, T_ID, SC_C: the correspondent DH-HMAC-CHAP parameters
When the host transmits the DH -HMAC-CHAP_Access-Request PDU, the parameters are instantiated as
follows:
• Responder’s Role: ‘C’
• Sequence Number: S2
• Augmented Challenge Value: Ca2
• Response Value: R2
• NQNR: NQNc
• HashID, T_ID, SC_C: the correspondent DH-HMAC-CHAP parameters
Upon receiving a DH -HMAC-CHAP_Access-Request PDU, the AVE shall perform the following steps in
order:
1. Lookup the DH-HMAC-CHAP key of the responder (i.e., Kr) from NQNR;
2. If the Responder’s Role is ‘H’, compute the expected response R’ as:
R’ = HMAC(Kr, Ca || S || T_ID || SC_C || ”HostHost” || NQNr || 00h || NQNa)
where NQNa is the NQN of the authenticator;
3. If the Responder’s Role is ‘C’, compute the expected response R’ as:
R’ = HMAC(Kr, Ca || S || T_ID || SC_C || ”Controller” || NQNr || 00h || NQNa)
where NQNa is the NQN of the authenticator; and
4. Compare the expected response R’ with the response value R received in the DH -HMAC-
CHAP_Access-Request PDU. If R’ = R then the authentication is successful; if R’ ≠ R then the
authentication has failed.
