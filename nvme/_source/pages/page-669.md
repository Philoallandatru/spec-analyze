---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 669
headings: ["8.3.4.4.2 AUTH_Failure Messages"]
---

# Source page 669

NVM Express® Base Specification, Revision 2.1

647
Figure 733: Authentication protocol identifiers
Value Definition
01h DH-HMAC-CHAP (refer to section 8.3.4.5)
All other values Reserved
Upon receiving an AUTH_Negotiate message, if the SC_C value indicated by the host:
• does not satisfy the security requirements of the controller (e.g., the host did not request secure
channel concatenation, but the controller’s security configuration requires secure channel
concatenation);
• is prohibited for the Admin Queue or I/O Queue via which the AUTH_Negotiate message was
received, as specified in this section; or
• requests secure channel concatenation and that value is contained in an AUTH_Negotiate
message received over a secure channel established in compliance with the requirements of an
NVMe transport specification (e.g., the NVM Express TCP Transport Specificat ion),
then the controller shall:
• reply to the AUTH_Negotiate message with an AUTH_Failure1 message having reason code
‘Authentication failure’ and reason code explanation ‘Secure channel concatenation mismatch’; and
• disconnect the NVMe over Fabrics connection upon transmitting the AUTH_Failure1 message.
Upon receiving an AUTH_Negotiate message, if the protocol descriptors proposed by the host do not satisfy
the security requirements of the controller, then the controller shall:
• reply to the AUTH_Negotiate message with an AUTH_Failure1 message having reason code
‘Authentication failure’ and reason code explanation ‘Authentication protocol not usable’; and
• disconnect the NVMe over Fabrics connection upon transmitting the AUTH_Failure1 message.
8.3.4.4.2 AUTH_Failure Messages
The AUTH_Failure1 message is sent from the controller to the host, the AUTH_Failure2 message is sent
from the host to the controller. The format of the AUTH_Failure1 message and of the AUTH_Failure2
message is shown in Figure 734.
Figure 734: AUTH_Failure1 and AUTH_Failure2 message format
Bytes Description
0 Authentication Type (AUTH_TYPE): 00h (i.e., common messages)
1
Authentication Identifier (AUTH_ID):
F0h (i.e., AUTH_Failure2)
F1h (i.e., AUTH_Failure1)
3:2 Reserved
5:4 Transaction Identifier (T_ID): 16-bit transaction identifier
6 Reason Code (RCODE)
7 Reason Code Explanation (RCODEEX)
The AUTH_Failure reason codes are listed in Figure 735.
Figure 735: AUTH_Failure reason codes
Value Definition
01h Authentication failure: The authentication transaction failed
All other values Reserved
The AUTH_Failure reason code explanations are listed in Figure 736.
