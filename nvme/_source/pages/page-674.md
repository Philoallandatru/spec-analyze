---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 674
headings: ["8.3.4.5.3 DH-HMAC-CHAP_Challenge Message"]
---

# Source page 674

NVM Express® Base Specification, Revision 2.1

652
The 00h identifier indicates that no Diffie-Hellman exchange is performed, which reduces the DH -HMAC-
CHAP protocol to the HMAC -CHAP protocol. The 00h identifier shall not be proposed in an
AUTH_Negotiate message that requests secure channel concatenation (i.e., with the SC_C field set to a
non-zero value).
The 2048-bit DH group and the 3072 -bit DH group shall be supported. A mechanism shall be provided to
disable (i.e., prohibit) use of the 2048-bit DH group. Use of the 2048-bit DH group may be prohibited by the
requirements of security policies that are not  defined by NVM Express (e.g., CNSA 1.0 requires use of a
3072-bit or larger DH group).
Upon receiving an AUTH_Negotiate message, if the DHgIDList proposed by the host:
• does not satisfy the security requirements of the controller (e.g., the host proposed only the NULL
DH group, but the controller’s security policy requires a DH group whose size is 3072-bit or larger);
or
• contains the NULL DH group (i.e., identifier 00h) and the AUTH_Negotiate message is requesting
secure channel concatenation (i.e., with the SC_C field set to a non-zero value),
then the controller shall:
• reply to the AUTH_Negotiate message with an AUTH_Failure1 message having reason code
‘Authentication failure’ and reason code explanation ‘DH group not usable’; and
• disconnect the NVMe over Fabrics connection upon transmitting the AUTH_Failure1 message.
8.3.4.5.3 DH-HMAC-CHAP_Challenge Message
The DH-HMAC-CHAP_Challenge message is sent from the controller to the host. The format of the DH -
HMAC-CHAP_Challenge message is shown in Figure 742.
Figure 742: DH-HMAC-CHAP_Challenge message format
Bytes Description
0 Authentication Type (AUTH_TYPE): 01h (i.e., DH-HMAC-CHAP)
1 Authentication Identifier (AUTH_ID): 01h (i.e., DH-HMAC-CHAP_Challenge)
3:2 Reserved
5:4 Transaction Identifier (T_ID): 16-bit transaction identifier
6 Hash Length (HL): Length in bytes of the selected hash function
7 Reserved
8 Hash Identifier (HashID): Identifier of selected hash function
9 Diffie-Hellman Group Identifier (DHgID): Identifier of selected Diffie-Hellman group
11:10 DH Value Length (DHVLEN): Length in bytes of DH value. If no DH value is included
in the message, then this field is cleared to 0h
15:12 Sequence Number (SEQNUM): Sequence number S1
15+HL:16 Challenge Value (CVAL): Challenge C1
15+HL+DHVLEN:16+HL DH Value (DHV): DH exponential gx mod p. This field is not present (i.e., the CVAL field
is the last field in the message) if DHVLEN is cleared to 0h
Hash Length (HL): Shall be set to the length in bytes of the selected hash function, as specified in Figure
740.
HashID: Shall be set to the hash function identifier (refer to Figure 740) selected for this authentication
transaction among those proposed in the DH -HMAC-CHAP protocol descriptor in the AUTH_Negotiate
message. The controller shall select a hash function in accord with its applicable policy.
DHgID: Shall be set to the DH group identifier (refer to Figure 742) selected for this authentication
transaction among those proposed in the DH -HMAC-CHAP protocol descriptor in the AUTH_Negotiate
message. The controller shall select a DH group identifier in accord with its applicable policy. If this field is
cleared to 0 h, then the DH portion of the DH -HMAC-CHAP protocol shall not be performed in this
authentication transaction. The controller shall not clear this field to 00h if the AUTH_Negotiate message
