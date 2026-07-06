---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 689
headings: ["8.3.4.6.4 AVE Discovery"]
---

# Source page 689

NVM Express® Base Specification, Revision 2.1

667
The NQN of the authenticator (i.e., NQN a) is retrieved from the TLS identity associated to the TLS
connection with the AVE (refer to section 8.3.4.6.2).
The result of an authentication verification is returned to the NVMe entity in a DH -HMAC-CHAP_Access-
Result PDU. The format of the DH-HMAC-CHAP_Access-Result PDU is shown in Figure 756.
Figure 756: DH-HMAC-CHAP_Access-Result PDU format
Bytes Description
00 PDU Type (PDU-Type): AFh for DH-HMAC-CHAP_Access-Result
01 Flags (FLAGS): Reserved
02 Header Length (HLEN): Fixed length of 8 bytes (08h)
03 PDU Data Offset (PDO): Reserved
07:04 PDU Length (PLEN): Fixed length of 20 bytes (14h)
15:08 Identifier (ID): 64-bit identifier used to match Access-Request and Access-Result PDUs
16
Authentication Verification Result (AuthRes):
Value Definition
01h Authentication Verification Successful
02h Authentication Verification Failed
All other values Reserved

17
Reason Code (RCODE): Additional explanation when authentication verification failed
Value Definition
00h No additional explanation
01h Authentication failure
02h Selected hash function unusable
All other values Reserved

19:18 Reserved
8.3.4.6.4 AVE Discovery
The AVE transport addresses may be configured on an NVMe entity or may be discovered by interacting
with a Discovery Controller (e.g., a CDC). An NVMe entity should randomly select any of the discovered
AVE transport addresses to connect to the AVE. The AV E Discovery log page is defined to facilitate this
discovery (refer to section 5.1.12.3.3).
