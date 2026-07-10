---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 677
headings: ["8.3.4.5.5 DH-HMAC-CHAP_Success1 Message"]
---

# Source page 677

NVM Express® Base Specification, Revision 2.1

655
with the same key may reveal information about the key or the correct response to this challenge. The
algorithm for generating the challenge value is outside the scope of this specification. Randomness of the
challenge value is crucial to the security of the protocol (refer to section 8.3.4.5.7). The CVAL length is the
same as the length of the selected hash function (i.e., HL).
DH Value (DHV):  Diffie-Hellman exponential. If the DH Value Length is cleared to 0h, this field is not
present. The DH Value shall be set to the value of g y mod p, where y is a random number selected by the
host that shall be at least 256 bits long (refer to section 8.3.4.5.7) and p and g shall have the values indicated
in Figure 741, based on the selected DH group identifier.
Upon receiving a DH-HMAC-CHAP_Reply message, if:
• the Hash Length (HL) does not match the value specified in Figure 740 for the selected hash
function;
• DHgID is non-zero and the DH Value Length (DHVLEN) is cleared to 0h; or
• DHgID is non-zero and the DH Value (DHV) is 0, 1, or p-1;
then the controller shall:
• reply with an AUTH_Failure1 message having reason code ‘Authentication failure’ and reason code
explanation ‘Incorrect payload’; and
• disconnect the NVMe over Fabrics connection.
In addition, the controller shall:
• check the challenge value C 2, if the Challenge Valid field is set to 01h, to verify it is different from
the challenge value C1 the controller previously sent. If C2 is equal to C1, the controller shall:
o reply with an AUTH_Failure1 message having reason code ‘Authentication failure’ and
reason code explanation ‘Authentication failed’; and
o disconnect the NVMe over Fabrics connection; and
• verify the response value R1 using the negotiated hash function. If verification of the response value
R1 does not succeed, the controller shall:
o reply with an AUTH_Failure1 message having reason code ‘Authentication failure’ and
reason code explanation ‘Authentication failed’; and
o disconnect the NVMe over Fabrics connection.
If verification of the response value R 1 succeeds, the host has been authenticated and the controller
shall continue with a DH-HMAC-CHAP_Success1 message.
8.3.4.5.5 DH-HMAC-CHAP_Success1 Message
The DH-HMAC-CHAP_Success1 message is sent from the controller to the host and indicates that the
controller has successfully authenticated the host. The format of the DH-HMAC-CHAP_Success1 message
is shown in Figure 744.
Figure 744: DH-HMAC-CHAP_Success1 message format
Bytes Description
0 Authentication Type (AUTH_TYPE): 01h (i.e., DH-HMAC-CHAP)
1 Authentication Identifier (AUTH_ID): 03h (i.e., DH-HMAC-CHAP_Success1)
3:2 Reserved
5:4 Transaction Identifier (T_ID): 16-bit transaction identifier
6 Hash Length (HL): Length in bytes of the selected hash function
7 Reserved
