---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 691
headings: ["9.6 Communication Loss Handling", "9.6.1 Host Communication Loss with a Controller"]
---

# Source page 691

NVM Express® Base Specification, Revision 2.1

669
software that a serious error condition has occurred.  When this condition occurs, host software should
attempt to reset and then re-initialize the controller.
The Controller Fatal Status condition is not indicated with an interrupt. If host software experiences timeout
conditions and/or repeated errors, then host software should consult the Controller Fatal Status
(CSTS.CFS) bit to determine if a more serious error has occurred.
If the Controller Fatal Status (CSTS.CFS) bit is set to ‘1’ on any controller in the NVM subsystem, the host
should issue a Controller Reset to that controller.
If that Controller Reset does not clear the Controller Fatal Status condition, the host should initiate an NVM
Subsystem Reset (refer to section 3.7.1), if supported.
Performing an NVM Subsystem Reset (NSSR) may cause PCI Express links to go down as part of resetting
the NVM subsystem. Host software may have undesirable effects related to PCI Express links going down
(e.g., some host operating systems or hypervisors may crash).
NVM Subsystem Reset should not be used if the host software has undesirable effects related to PCI
Express links going down. This host software includes, but is not limited to, operating systems using
Firmware First Error Handling (refer to the ACPI Specification). Such operating systems should not use
NSSR for recovery from CFS conditions.
9.6 Communication Loss Handling
If the host loses communication with a controller, then the host is unable to receive a completion (CQE) for
any outstanding command that has been submitted to that controller (refer to section 3.4.5). If the host is
able to use another controller to access the same NVM subsystem or re -establish communication with the
original controller, then it is strongly recommended that any host use of that controller to recover from
communication loss follow the  procedures and requirements in this section in order to avoid possible
corruption of user data and unintended changes to NVM subsystem state.
Host recovery from communication loss with a controller consists of three functional components :
• Host determination that communication with a controller has been lost is described in section 9.6.1.
• Host determination that no further processing of outstanding commands is possible on that
controller is described in section 9.6.2.
• Host retry, if any, of outstanding commands after communication loss is described in section 9.6.3.
These functional components interact with each other. Host detection of communication loss is necessary
before the host is able to determine when no further controller processing of outstanding commands is
possible. Host retries of outstanding commands that modify user data or NVM subsystem state are able to
corrupt user data or make unintended changes to NVM subsystem state unless the host determines that
no further controller processing of the original commands is possible as described in section 9.6.2.
 Host Communication Loss with a Controller
A host determines that communication has been lost with a controller if:
• the host detects a Keep Alive Timeout (refer to section 3.9);
• for message-based transports, the host or the controller terminates the NVMe Transport connection
on which the command was sent (refer to section 3.3.2.4); or
• the host detects a transport connection loss using methods outside the scope of this specification
(e.g., the transport notifies the host of a loss of communication either with the controller to which
the command was submitted or with the queue on which the command was sent).
A controller may detect a loss of communication at a different time (e.g., later) than the host detects that
loss of communication. As explained in section 9.6.2, additional time may be required for the controller to
stop processing commands after the controller detects a loss of communication.
