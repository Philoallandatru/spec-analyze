---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 676
headings: []
---

# Source page 676

NVM Express® Base Specification, Revision 2.1

654
Figure 743: DH-HMAC-CHAP_Reply message format
Bytes Description
8
Challenge Valid (CVALID):
Value Definition
00h The Challenge Value is not valid
01h The Challenge Value is valid
All other values Reserved

9 Reserved
11:10 DH Value Length (DHVLEN):  Length in bytes of DH value. If no DH value is
included in the message, then this field is cleared to 0h.
15:12 Sequence Number (SEQNUM): Sequence number S2.
15+HL:16 Response Value (RVAL): Response R1.
15+2*HL:16+HL Challenge Value (CVAL): Challenge C2, if valid (i.e., if the CVALID field is set to
01h), cleared to 0h otherwise.
15+2*HL+DHVLEN:6+2*HL DH Value (DHV):  DH exponential g y mod p. This field is not present (i.e., the
CVAL field is the last field in the message) if DHVLEN is cleared to 0h.
Hash Length (HL): Shall be set to the length in bytes of the selected hash function, as specified in Figure
740.
Challenge Valid: If the host does not require bidirectional authentication or no establishment of a secure
channel after unidirectional authentication is sought (refer to section 8.3.4.5.9), this field shall be cleared to
0h. Otherwise, this field shall be set to 01h.
DH Value Length (DHVLEN): Diffie-Hellman exponential length. This length shall be a multiple of 4. If the
DH group identifier is cleared to 0h (i.e., NULL DH exchange), this field shall be cleared to 0h. Otherwise,
it shall be set to the length in bytes of the DH Value.
Sequence Number (SEQNUM): 32-bit sequence number S2. A random non-zero value shall be used as
the initial value. The sequence number is incremented modulo 232 after each use, except that the value 0h
is skipped (i.e., incrementing the value FFFFFFFFh results in the value 00000001h). The value 0h is used
to indicate that bidirectional authentication is not performed, but a challenge value C 2 is carried in order to
generate a pre -shared key (PSK) for subsequent establishment of a secure channel (refer to sect ion
8.3.4.5.9).
Response Value (RVAL):  DH-HMAC-CHAP response value R 1. The value of R 1 is computed using the
hash function H( ) selected by the HashID parameter in the DH -HMAC-CHAP_Challenge message, and
the augmented challenge C a1. If the NULL DH group has been selected, the augmented challenge C a1 is
equal to the challenge C 1 received from the controller (i.e., C a1 = C1). If a non -NULL DH group has been
selected, the augmented challenge is computed applying the HMAC function using the hash function H( )
selected by the H ashID parameter in the DH -HMAC-CHAP_Challenge message with the hash of the
ephemeral DH key resulting from the combination of the random value y selected by the host with the DH
exponential (i.e., gx mod p) received from the controller as HMAC key (refer to RFC 2104) to the challenge
C1 (i.e., C a1 = HMAC(H((g x mod p) y mod p), C 1) = HMAC(H(g xy mod p), C 1)). The value of R 1 shall be
computed applying the HMAC function using the hash function H( ) selected by the HashID parameter in
the DH -HMAC-CHAP_Challenge message with key K h as HMAC key to the concatenation of the
augmented challenge C a1, the sequence number S 1, the transaction identifier T_ID, the secure channel
concatenation indication SC_C sent in the AUTH_Negotiate message, the eight ASCII characters
”HostHost” to indicate the host is computing the reply, the host NQN not including the null terminator, a 00h
byte, and the NVM subsystem NQN not including the null terminator (i.e., R 1 = HMAC(Kh, Ca1 || S1 || T_ID
|| SC_C || ”HostHost” || NQNh || 00h || NQNc)). Using C language notation:
Ca1 = (DHgID == 00h) ? C1 : HMAC(H((gx mod p)y mod p)), C1)
R1 = HMAC(Kh, Ca1 || S1 || T_ID || SC_C || ”HostHost” || NQNh || 00h || NQNc)
Challenge Value (CVAL):  Shall be set to a random challenge value C 2 (refer to section 8.3.4.5.7). Each
challenge value should be unique and unpredictable, since repetition of a challenge value in conjunction
