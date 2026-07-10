---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 309
headings: []
---

# Source page 309

NVM Express® Base Specification, Revision 2.1

287
Figure 294: Discovery Log Page Entry Data Structure
Bytes Description
02
Subsystem Type (SUBTYPE): Specifies the type of the NVM subsystem that is indicated in this
entry.
Value Definition
00 Reserved
01
Referral: The entry describes a referral to another Discovery Service
composed of Discovery controllers that provide additional discovery records.
Multiple Referral entries may be reported for each Discovery Service (e.g., if
that Discovery Service has multiple NVM subsystem ports or supports multiple
protocols).
02
NVM Subsystem: The entry describes an NVM subsystem whose controllers
may have attached namespaces (i.e., an NVM subsystem that is not composed
of Discovery controllers). Multiple NVM Subsystem entries may be reported for
each NVM subsystem if that NVM subsystem has mul tiple NVM subsystem
ports.
03
Current Discovery Subsystem:  The entry describes this Discovery
subsystem (i.e., the Discovery Service that contains the controller processing
the Get Log Page command). Multiple Current Discovery Subsystem entries
may be reported for this Discovery subsystem if the current Discovery
subsystem has multiple NVM subsystem ports.
04 to 255 Reserved

03
Transport Requirements (TREQ): Indicates requirements for the NVMe Transport.
Bits Description
7:6 Reserved
5:4
Transport Authentication and Secure Channel (TASC): Indicates whether an
authentication transaction (refer to section 8.3.4.2) or an authentication transaction
concatenated to a secure channel establishment (refer to section 8.3.4.3) is required
upon completion of the Connect command. These bits do not override the AUTHREQ
bits in the Connect response; these bits apply only when the AUTHREQ bits specify
no requirements (refer to Figure 548).
Value Definition
00b Not specified
01b Authentication required
10b Authentication concatenated to secure channel establishment required
11b Reserved

3
Zero Host ID Support (ZHIDS): If this bit is set to ‘1’, then the controller supports a
Host Identifier value of 0h in a Connect command. If this bit is cleared to ‘0’, then the
controller does not support a Host Identifier value of 0h in a Connect command.
2
SQ Flow Control Disable (SQFCD): If this bit is set to ‘1’ , then  the controller is
capable of disabling SQ flow control. A controller that is capable of disabling SQ flow
control may accept or reject a host request to disable SQ flow control. If this bit is
cleared to ‘0’, then the controller requires use of SQ flow control.
1:0
Transport Secure Channel (TSC): Indicates whether connections shall be made
over a fabric secure channel (refer to section 8.3.4.1).
Value Definition
00b Not specified
01b Required
10b Not required (i.e., not supported or supported but not required)
11b Reserved


05:04
Port ID (PORTID): Specifies a particular NVM subsystem port. Different NVMe Transports or
address families may utilize the same Port ID value (e.g., a Port ID may support both iWARP and
RoCE).
