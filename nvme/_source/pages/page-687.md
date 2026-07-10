---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 687
headings: ["8.3.4.6.2 AVE Connections", "8.3.4.6.3 AVE Access Protocol"]
---

# Source page 687

NVM Express® Base Specification, Revision 2.1

665
8.3.4.6.2 AVE Connections
An NVMe entity (i.e., a host or a controller) connection with a DH-HMAC-CHAP AVE shall use TLS version
1.3 (refer to RFC 8446) with pre-shared key (PSK) authentication, as specified for NVMe/TCP (refer to the
NVM Express TCP Transport Specification).
In order to establish a TLS connection with an AVE, an NVMe entity requires a PSK shared between that
entity and the AVE (i.e., PSKea) for authentication of the TLS connection. PSKea shall be either derived from
the DH-HMAC-CHAP secret, if the DH-HMAC-CHAP secret representation specifies a hash function (refer
to section 8.3.4.5.8), or provisioned on the NVMe entity through a configured PSK, as specified for
NVMe/TCP.
Derivation of PSKea from a DH-HMAC-CHAP secret shall use the HKDF-Extract and HKDF-Expand-Label
operations (refer to RFC 5869 and RFC 8446) in the following order:
1. PRK = HKDF-Extract(0, DH-HMAC-CHAP secret); and
2. PSKea = HKDF -Expand-Label(PRK, “A -V-Entity” || NQN AVE, NQN e, Length( DH-HMAC-CHAP
secret)),
where NQNAVE is the NQN of the AVE and NQN e is the NQN of the NVMe entity. The hash function used
with HKDF shall be the one specified in the DH -HMAC-CHAP secret representation (refer to section
8.3.4.5.8). This transform requires that the NVMe entity knows NQN AVE.
Derivation of PSKea from a DH-HMAC-CHAP secret is not possible if the DH-HMAC-CHAP secret
representation does not specify a hash function. In this case PSKea shall be provisioned on the NVMe entity
through a configured PSK, as specified for NVMe/TCP, and that configured PSK should be different than
the DH-HMAC-CHAP secret for that entity.
Derivation of PSKea from a configured PSK shall use the HKDF-Extract and HKDF-Expand-Label operations
in the following order:
1. PRK = HKDF-Extract(0, Configured PSK); and
2. PSKea = HKDF-Expand-Label(PRK, “A-V-Entity” || NQNAVE, NQNe, Length(Configured PSK)),
where NQNAVE is the NQN of the AVE and NQN e is the NQN of the NVMe entity. The hash function used
with HKDF shall be the one specified in the PSK interchange format (refer to the NVM Express TCP
Transport Specification). If no hash function is specified in the PSK interchange format, then the configured
PSK shall be used as PSKea. This transform requires that the NVMe entity knows NQN AVE.
The TLS connection with the AVE shall be performed as specified in the TLS PSK and PSK Identity
Derivation section of the NVM Express TCP Transport Specification, with PSKea used as the Retained PSK,
NQNe used as the host NQN, and NQN AVE used as the controller NQN. Mandatory and recommended
cipher suites for this TLS connection are specified in the Mandatory and Recommended Cipher Suites
section of the NVM Express TCP Transport Specification. This TLS connection may be used for multiple
authentication verif ications. An NVMe entity may terminate this TLS connection and re -establish it as
required. An AVE may terminate this TLS connection after some period of inactivity (e.g., 10 minutes). An
NVMe entity may avoid termination of this TLS connection by using th e TLS heartbeat extension (refer to
RFC 6520).
8.3.4.6.3 AVE Access Protocol
Communication with a DH -HMAC-CHAP AVE uses two PDUs, DH -HMAC-CHAP_Access-Request and
DH-HMAC-CHAP_Access-Result, that are sent directly over the TLS connection with the AVE (refer to
section 8.3.4.6.2). The format of the DH-HMAC-CHAP_Access-Request PDU is shown in Figure 755.
Figure 755: DH-HMAC-CHAP_Access-Request PDU format
Bytes Description
00 PDU Type (PDU-Type): AEh for DH-HMAC-CHAP_Access-Request
01 Flags (FLAGS): Reserved
02 Header Length (HLEN): Fixed length of 8 bytes (08h)
