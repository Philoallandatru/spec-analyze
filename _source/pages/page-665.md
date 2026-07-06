---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 665
headings: ["8.3.4.2.2 NVMe In-band Authentication Protocol-Specific Requirements"]
---

# Source page 665

NVM Express® Base Specification, Revision 2.1

643
transaction after an authentication transaction timeout should be aborted with a status code of Command
Sequence Error.
Figure 727 shows an example of authentication transaction for NVMe/TCP.
Figure 727: Example of authentication transaction for NVMe/TCP
1. An NVMe/TCP transport session is established
2. The Connect exchange is performed to set up an
NVMe Queue and associate host to controller
3. The host performs an authentication transaction
with the controller to authenticate the end-points
4. Queue is set up, ready for subsequent operations
Host Controller
Queue set up
Authentication Transaction
Connect Command
Connect Response
NVMe/TCP transport session establishment

8.3.4.2.2 NVMe In-band Authentication Protocol-Specific Requirements
Authentication requirements for security commands are based on the security protocol indicated by the
SECP field in the command.
The authentication protocols defined by this specification use the security protocol identifier E9h (assigned
to NVMe by SPC -5, a SCSI standard). The messages of the defined authentication protocols are self -
identifying, therefore the SPSP0 field and the S PSP1 field of the Authentication Send and Authentication
Receive commands shall be set to 01h. Authentication messages are mapped to NVMe over Fabrics
command and response pairs. The mapping of authentication messages to the Authentication Send
command is shown in Figure 728.
Figure 728: Mapping of authentication messages to the Authentication Send command
Field1 Value
SPSP0 01h
SPSP1 01h
SECP E9h
TL Specifies the amount of data to transfer in bytes
Notes:
1. Refer to section 6.2.
The mapping of authentication messages to the Authentication Receive command is shown in Figure 729.
Security processing requirements associated with the Authentication Receive command (e.g., delays in
third-party authentication verification) may result in delays in controller completion of an Authentication
Receive command. The host should consider these possible delays associated with the Authentication
Receive command.
