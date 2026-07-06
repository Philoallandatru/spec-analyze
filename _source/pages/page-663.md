---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 663
headings: ["8.3.4.2 NVMe In-band Authentication", "8.3.4.2.1 Special considerations for In-band Authentication of Discovery subsystems"]
---

# Source page 663

NVM Express® Base Specification, Revision 2.1

641
 NVMe In-band Authentication
The Authentication and Security Requirements (AUTHREQ) field in the Connect response capsule (refer
to Figure 547) indicates whether NVMe in-band authentication is required.
If one or more of the bits in the AUTHREQ field are set to ‘1’, then the controller requires that the host
authenticate on that queue in order to proceed with Fabrics, Admin, and I/O commands. Authentication
success is defined by the specific security prot ocol that is used for authentication. If any command other
than Connect, Authentication Send, or Authentication Receive is received prior to authentication success,
then the controller shall abort the command with Authentication Required status.
If all bits in the AUTHREQ field are cleared to ‘0’, then the controller does not require the host to
authenticate, and the NVM subsystem shall not abort any command with a status code value of
Authentication Required.
Refer to section 8.3.4.2.1 for considerations on Discovery subsystems.
8.3.4.2.1 Special considerations for In-band Authentication of Discovery subsystems
Hosts that have been configured to authenticate Discovery subsystems with an in -band authentication
protocol that supports both unidirectional authentication and bidirectional authentication (e.g., DH -HMAC-
CHAP, refer to section 8.3.4.5) should behave as follows:
• If the host connected to a Discovery subsystem using the well-known Discovery Service NQN (i.e.,
nqn.2014-08.org.nvmexpress.discovery) and the Discovery subsystem did not request
authentication, then the host should not perform an authentication transaction;
• If the host connected to a Discovery subsystem using the well-known Discovery Service NQN (i.e.,
nqn.2014-08.org.nvmexpress.discovery) and the Discovery subsystem requested authentication,
then the host should perform only unidirectional authentication (i.e., the Discovery subsystem may
authenticate the host, but the host should not authenticate the well-known Discovery Service NQN);
or
• If the host connected to a Discovery subsystem using the unique Discovery Service NQN for that
Discovery subsystem (refer to section 3.1.3.3), regardless of whether the Discovery subsystem
requested authentication, then the host may perform unidirectional authentication or bidirectional
authentication (i.e., the host may authenticate the unique Discovery Service NQN for that Discovery
subsystem).
Figure 726 illustrates a process that a host is able to use to retrieve the unique Discovery Service NQN and
perform in-band authentication for a Discovery subsystem if that host has been configured to authenticate
Discovery subsystems and the Discovery subsystem that the host connects to also requires authentication
(i.e., the AUTHREQ field is not cleared to zero) using DH-HMAC-CHAP for authentication . This process
includes:
1. connect to the Discovery subsystem using the well-known Discovery Service NQN (i.e., nqn.2014-
08.org.nvmexpress.discovery);
2. perform unidirectional authentication with the Discovery subsystem;
3. perform controller initialization (refer to section 3.5);
4. retrieve the unique Discovery Service NQN (refer to section 3.1.3.3);
5. perform controller shutdown (refer to section 3.6);
6. reconnect to the Discovery subsystem using the unique Discovery Service NQN for that Discovery
subsystem; and
7. perform bidirectional authentication with the Discovery subsystem using the unique Discovery
Service NQN for that Discovery subsystem.
