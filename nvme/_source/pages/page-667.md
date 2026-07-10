---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 667
headings: ["8.3.4.4 Common Authentication Messages", "8.3.4.4.1 AUTH_Negotiate Message"]
---

# Source page 667

NVM Express® Base Specification, Revision 2.1

645
Figure 730: Example of TLS secure channel concatenation with NVMe/TCP
Host Controller
1. An NVMe/TCP transport session is established
2. The Connect exchange is performed to set up an
Admin Queue and associate host to controller
3. An authentication transaction generating a PSK
between host and controller is performed
4. The NVMe/TCP transport session is disconnected
5. An NVMe/TCP-TLS transport session is established
using the generated PSK
6. The Connect exchange is performed to set up an
Admin Queue and associate host to controller
7. Secure channel and Queue are set up
8. An NVMe/TCP-TLS transport session is established
using the generated PSK
9. The Connect exchange is performed to set up the
first I/O Queue of the established association
10. Secure channel and Queue are set up
11. An NVMe/TCP-TLS transport session is established
using the generated PSK
12. The Connect exchange is performed to set up the
Nth I/O Queue of the established association
13. Secure channel and Queue are set up
Connect Command
Connect Response
Secure channel and Admin Queue set up
NVMe/TCP transport session disconnect
NVMe/TCP-TLS transport session establishment with PSK
Connect Command
Connect Response
NVMe/TCP-TLS transport session establishment with PSK
Connect Command
Connect Response
Secure channel and 1st I/O Queue set up
NVMe/TCP-TLS transport session establishment with PSK
Connect Command
Connect Response
Secure channel and Nth I/O Queue set up
PSKPSK

Secure channel concatenation is prohibited over any secure channel that has been established in
compliance with the requirements of an NVMe transport specification (e.g., the NVM Express TCP
Transport Specification). The controller response to a host that requests secure channel concatenation in
this situation is specified in section 8.3.4.4.1. In contrast, replacing a PSK for such a secure channel is
permitted (refer to section 8.3.4.4.1.
 Common Authentication Messages
8.3.4.4.1 AUTH_Negotiate Message
The AUTH_Negotiate message is sent from the host to the controller and is used to indicate the
authentication protocols the host is able to use in this authentication transaction and which secure channel
protocol, if any, to concatenate to this authentication transaction. The AUTH_Negotiate message format is
shown in Figure 731.
