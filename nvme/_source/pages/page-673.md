---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 673
headings: []
---

# Source page 673

NVM Express® Base Specification, Revision 2.1

651
Figure 739: Authentication protocol descriptor for DH-HMAC-CHAP
Bytes Description
0 Authentication Protocol Identifier (AuthID): (01h for DH-HMAC-CHAP)
1 Reserved
2 HashIDList Length (HALEN): Number of hash function identifiers (1 to 30)
3 DHgIDList Length (DHLEN): Number of Diffie-Hellman group identifiers (1 to 30)
3+HALEN:4 Hash Function Identifier List (HashIDList ): Array of hash function identifiers (one byte per
identifier)
33:4+HALEN Pad (PAD): Padding bytes cleared to 0h, if present
33+DHLEN:34 Diffie-Hellman Group Identifier List (DHgIDList): Array of Diffie-Hellman Group identifiers (one
byte per identifier)
63:34+DHLEN Pad1 (PAD1): Padding bytes cleared to 0h, if present
The one-way hash functions used by DH-HMAC-CHAP are shown in Figure 740.
Figure 740: DH-HMAC-CHAP hash function identifiers
Identifier Hash Function Hash Length (bytes) Hash Block Size1 (bytes) Reference
00h Reserved
01h SHA-256 32 64 RFC 6234
02h SHA-384 48 128 RFC 6234
03h SHA-512 64 128 RFC 6234
04h-DFh Reserved
E0h-FEh Vendor specific
FFh Reserved
Notes:
1. The hash block size is used by the HMAC calculation
The SHA-256 hash function shall be supported. Use of the SHA -256 hash function may be prohibited by
the requirements of security policies that are not defined by NVM Express (e.g., CNSA 1.0 requires use of
SHA-384).
Upon receiving an AUTH_Negotiate message, if the HashIDList proposed by the host does not satisfy the
security requirements of the controller (e.g., the host proposed SHA-256, but the controller’s security policy
requires a SHA-384 hash), then the controller shall:
• reply to the AUTH_Negotiate message with an AUTH_Failure1 message having reason code
‘Authentication failure’ and reason code explanation ‘Hash function not usable’; and
• disconnect the NVMe over Fabrics connection upon transmitting the AUTH_Failure1 message.
The Diffie-Hellman (DH) groups used by DH-HMAC-CHAP are shown in Figure 741.
Figure 741: DH-HMAC-CHAP Diffie-Hellman group identifiers
Identifier DH group size Generator (g) Modulus (p) and Reference
00h NULL n/a n/a
01h 2048-bit 2 refer to RFC 7919
02h 3072-bit 2 refer to RFC 7919
03h 4096-bit 2 refer to RFC 7919
04h 6144-bit 2 refer to RFC 7919
05h 8192-bit 2 refer to RFC 7919
06h-DFh Reserved
E0h-FEh Vendor specific
FFh Reserved
