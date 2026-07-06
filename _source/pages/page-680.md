---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 680
headings: ["8.3.4.5.8 Secret Representation", "8.3.4.5.9 Generated PSK for TLS"]
---

# Source page 680

NVM Express® Base Specification, Revision 2.1

658
each host and controller is outside of scope of this specification. For instance, secrets could be provisioned
via an encrypted HTTPS-based connection.
8.3.4.5.8 Secret Representation
In order to facilitate provisioning, management, and interchange (e.g., copy & paste in an administrative
configuration tool) of secrets, all NVMe over Fabrics entities shall support the following ASCII representation
of secrets:
DHHC-1:xx:<Base64 encoded string>:
Where:
• ”DHHC-1” indicates this is a version 1 representation of a secret for the DH-HMAC-CHAP protocol;
• ‘:’ is used both as a separator and a terminator;
• xx indicates the hash function to be used to transform the secret in key (refer to section 8.3.4.5.7),
encoded as the ASCII representation of the hexadecimal value specified in Figure 740 (e.g., the
two ASCII characters “01” indicate SHA-256). The two ASCII characters “00” indicate no transform
(i.e., use the secret as a key); and
• The Base64 (refer to RFC 4648) string encodes the secret (32, 48, or 64 bytes binary) followed by
the CRC-32 (refer to RFC 1952) of the secret (4 bytes binary).
As an example, the 32-byte secret:
89AEB31A 874EAF84 841B4673 6B0DFDF2 BA58D30A A2A545A3 E235A352 1E07594Ch
has the CRC-32 A70D69FAh, that is represented in little endian format (i.e., FA690DA7h) for concatenation
to the secret, resulting in the following:
“DHHC-1:00:ia6zGodOr4SEG0Zzaw398rpY0wqipUWj4jWjUh4HWUz6aQ2n:”
when intended to be used as a key without transform.
When provided with a secret in this format, NVMe over Fabrics  entities shall verify the validity of the
provided secret by computing the CRC -32 value of the secret and checking the computed value with the
provided value. If they do not match, then the secret shall not be used.
8.3.4.5.9 Generated PSK for TLS
When used with a non -NULL DH exchange, the DH -HMAC-CHAP protocol is able to generate a session
key KS used to generate a pre-shared key (PSK) to establish a secure channel session with the TLS protocol
between host and controller for NVMe/TCP. A TLS session is concatenated to an authentication transaction
when the SC_C indication is set to NEWTLSPSK in the AUTH_Negotiate message (refer to section 8.3.4.1).
A TLS session shall not be concatenated to an authentication transaction if the involved host and controller
are administratively configured with a PSK for use with each other. In this case, host and controller shall
only establish a TLS session based on the retained PSK derived from that configured PSK.
The session key K S shall be computed from the ephemeral DH key (i.e., g xy mod p) generated during the
DH-HMAC-CHAP transaction by applying the hash function H( ) selected by the HashID parameter in the
DH-HMAC-CHAP_Challenge message (i.e., K S = H(g xy mod p)). The size of the session key K S is
determined by the selected hash function, as shown in Figure 740. Specifically:
• The host computes KS as the hash of the ephemeral DH key resulting from the combination of the
random value y selected by the host with the DH exponential (i.e., g x mod p) received from the
controller (i.e., KS = H((gx mod p)y mod p) = H(gxy mod p)).
• The controller computes K S as the hash of the ephemeral DH key resulting from the combination
of the random value x selected by the controller with the DH exponential (i.e., g y mod p) received
from the host (i.e., KS = H((gy mod p)x mod p) = H(gxy mod p)).
The generated PSK for TLS shall be computed applying the HMAC function using the hash function H( )
selected by the HashID parameter in the DH -HMAC-CHAP_Challenge message with the session key K S
as key to the concatenation of the two challenges C 1 and C2 (i.e., generated PSK = HMAC(K S, C1 || C2)).
