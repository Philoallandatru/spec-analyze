---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 664
headings: []
---

# Source page 664

NVM Express® Base Specification, Revision 2.1

642
Figure 726: Unique Discovery Service NQN retrieval for bidirectional authentication
Host Functionality
(Host or CDC)
Discovery
Controller (DC)
Transport connection established
Response (AUTHREQ = 1)
Connect (SUBNQN = well-known DS NQN)
Controller Initialization
Controller Shutdown
Transport connection terminated
Transport connection established
DH-HMAC-CHAP_Challenge
AUTH_Negotiate
DH-HMACH-CHAP_Success1
DH-HMAC-CHAP_Reply
Response (Unique DS NQN)
Identify (CNS 01h)
Response (AUTHREQ = 1)
Connect (SUBNQN = Unique DS NQN)
DH-HMAC-CHAP_Challenge
AUTH_Negotiate
DH-HMACH-CHAP_Success1
DH-HMAC-CHAP_Reply
DH-HMACH-CHAP_Success2
1
2
3
4
5
6
7
• Unidirectional authentication
• DC authenticates host
• Host does not authenticate DC
since it is only aware of the well -
known Discovery service NQN
• Bidirectional authentication
• DC authenticates host
• Host authenticates DC using
unique Discovery service NQN
discovered above

The host may initiate a subsequent authentication transaction at any time for reauthentication purposes.
Initiating reauthentication shall not invalidate a prior authentication. If the reauthentication transaction
concludes with the controller sending an A UTH_Failure1 message (refer to section 8.3.4.4.2), then the
controller shall terminate all commands with a status code of Operation Denied and disconnect the NVMe
over Fabrics connection . If the reauthentication transaction concludes with the host sending an
AUTH_Failure2 message, then the host shall disconnect the NVMe over Fabrics connection.
The state of an in -progress authentication transaction is soft -state. If the subsequent command in an
authentication transaction is not received by the controller within a timeout equal to:
• the Keep Alive Timeout value (refer to Figure 545), if the Keep Alive Timer is enabled; or
• the default Keep Alive Timeout value (i.e., two minutes), if the Keep Alive Timer is disabled;
then the authentication transaction has timed out and the controller should discard the authentication
transaction state (including the T_ID value, refer to section 8.3.4.4.1).
For an initial authentication, an authentication transaction timeout should be treated as an authentication
failure with termination of the transport connection. For reauthentication, an authentication transaction
timeout should not be treated as an authentication failure. Authentication commands used to continue that
