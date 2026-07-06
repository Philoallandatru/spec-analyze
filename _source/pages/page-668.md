---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 668
headings: []
---

# Source page 668

NVM Express® Base Specification, Revision 2.1

646
Figure 731: AUTH_Negotiate message format
Bytes Description
0 Authentication Type (AUTH_TYPE): 00h (i.e., common messages)
1 Authentication Identifier (AUTH_ID): 00h (i.e., AUTH_Negotiate)
3:2 Reserved
5:4 Transaction Identifier (T_ID): 16-bit transaction identifier
6 Transaction Identifier (SC_C)
7
Number of Authentication Protocol Descriptors ( NAPD): This field
specifies the number of Authent ication Protocol Descriptors are in the
Authentication Protocol Descriptor list.
Authentication Protocol Descriptor List
71:8 Authentication Protocol Descriptor 1
135:72 Authentication Protocol Descriptor 2
… …
NAPD*64+7:(NAPD-1)*64+8 Authentication Protocol Descriptor NAPD
The SC_C field determines if a secure channel concatenation to the authentication transaction is requested
and with which secure channel protocol, as shown in Figure 732.
Figure 732: Secure channel protocol identifiers
Value Definition Transport
Applicability
00h NOSC: No secure channel concatenation  n/a
01h Obsolete (refer to NVM Express Base Specification 2.0)
02h
NEWTLSPSK: Used on an Admin Queue over a TCP channel without TLS to
generate a PSK and associated PSK identity. This {PSK, PSK Identity} pair may be
used to set up TLS secure channels for subsequent Admin and I/O queues.
TCP
03h
REPLACETLSPSK: Used on an Admin Queue over a TLS secure channel to
generate a PSK and associated PSK identity. This {PSK, PSK Identity} pair replaces
the {PSK, PSK Identity} pair that was used to set up the TLS secure channel over
which the authentication transaction is performed.
TCP with TLS
All other
values Reserved
An authentication transaction with the SC_C field set to NOSC is allowed on any Admin or I/O Queue and
does not generate a PSK (i.e., it performs authentication only).
An authentication transaction with the SC_C field set to NEWTLSPSK:
• is allowed on an Admin Queue over a TCP channel without TLS and generates a new PSK;
• is prohibited on an I/O Queue over a TCP channel without TLS; and
• is prohibited on an Admin Queue or I/O queue over a TLS secure channel.
An authentication transaction with the SC_C field set to REPLACETLSPSK:
• is allowed on an Admin Queue over a TLS secure channel and replaces the association’s PSK;
• is prohibited on an Admin Queue or I/O queue over a TCP channel without TLS; and
• is prohibited on an I/O Queue over a TLS secure channel.
The AUTH_Negotiate message is structured as a list of 64 -byte authentication protocol descriptors to
enable extensibility to define additional authentication protocols. Currently only one authentication protocol
is defined (i.e., DH-HMAC-CHAP), therefore the AUTH_Negotiate message carries only one authentication
protocol descriptor (i.e., NAPD=1). Implementations should support more than one descriptor to enable
protocol extensibility. The first byte of an authentication protocol descriptor identifies the s pecific
authentication protocol, as shown in Figure 733.
