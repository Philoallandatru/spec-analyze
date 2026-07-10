---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 675
headings: ["8.3.4.5.4 DH-HMAC-CHAP_Reply Message"]
---

# Source page 675

NVM Express® Base Specification, Revision 2.1

653
(refer to section  8.3.4.4.1) for this instance of the DH -HMAC-CHAP protocol requested secure channel
concatenation (i.e., the SC_C field in that message was set to a non-zero value).
DH Value Length (DHVLEN): Diffie-Hellman exponential length. This length shall be a multiple of 4. If the
DH group identifier is cleared to 0h  (i.e., NULL DH exchange), this field shall be cleared to 0h. Otherwise,
it shall be set to the length in bytes of the DH Value.
Sequence Number (SEQNUM): 32-bit sequence number S1. A random non-zero value shall be used as
the initial value. The sequence number is incremented modulo 232 after each use, except that the value 0h
is skipped (i.e., incrementing the value FFFFFFFFh results in the value 00000001h).
Challenge Value (CVAL): Shall be set to a random challenge value C 1 (refer to section 8.3.4.5.7). Each
challenge value should be unique and unpredictable, since repetition of a challenge value in conjunction
with the same key may reveal information about the key or the correct response to this challenge. The
algorithm for generating the challenge va lue is outside the scope of this specification. Randomness of the
challenge value is crucial to the security of the protocol (refer to section 8.3.4.5.7). The CVAL length is the
same as the length of the selected hash function (i.e., HL).
DH Value (DHV): Diffie-Hellman exponential. If the DH Value Length is cleared to 0h, this field is not
present. The DH value shall be set to the value of g x mod p, where x is a random number selected by the
controller that shall be at least 256 bits long (refer to section 8.3.4.5.7) and p and g shall have the values
indicated in Figure 741, based on the selected DH group identifier.
Upon receiving a DH-HMAC-CHAP_Challenge message, if:
• the Hash Length (HL) does not match the value specified in Figure 740 for the selected hash
function;
• the Sequence Number (SEQNUM) is cleared to 0h;
• DHgID is cleared to 0h and the AUTH_Negotiate message (refer to section 8.3.4.4.1) that the host
sent for this instance of the DH -HMAC-CHAP protocol requested secure channel concatenation
(i.e., the SC_C field in that message is set to a non-zero value);
• DHgID is non-zero and the DH Value Length (DHVLEN) is cleared to 0h; or
• DHgID is non-zero and the DH Value (DHV) is 0, 1, or p-1;
then the host shall:
• reply with an AUTH_Failure2 message having reason code ‘Authentication failure’ and reason code
explanation ‘Incorrect payload’; and
• disconnect the NVMe over Fabrics connection.
8.3.4.5.4 DH-HMAC-CHAP_Reply Message
The DH -HMAC-CHAP_Reply message is sent from the host to the controller. The host may request
authentication of the controller to enable bidirectional authentication, by including a DH -HMAC-CHAP
challenge value C2 in this message. The challenge value C 2 shall be different from the challenge value C 1
received in the DH-HMAC-CHAP_Challenge message.
The format of the DH-HMAC-CHAP_Reply message is shown in Figure 743.
Figure 743: DH-HMAC-CHAP_Reply message format
Bytes Description
0 Authentication Type (AUTH_TYPE): 01h (i.e., DH-HMAC-CHAP)
1 Authentication Identifier (AUTH_ID): 02h (i.e., DH-HMAC-CHAP_Reply)
3:2 Reserved
5:4 Transaction Identifier (T_ID): 16-bit transaction identifier
6 Hash Length (HL): Length in bytes of the selected hash function
7 Reserved
