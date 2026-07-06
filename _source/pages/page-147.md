---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 147
headings: ["3.9.1 Keep Alive Timer Configuration", "3.9.2 Keep Alive Timer Activation", "3.9.3 Command Based Keep Alive", "3.9.3.1 Command Based Keep Alive on the Controller"]
---

# Source page 147

NVM Express® Base Specification, Revision 2.1

125
NVMe Transports that do not detect a connection loss in a timely manner shall require that the Keep Alive
Timer feature be supported and enabled.
 Keep Alive Timer Configuration
A host configures the Keep Alive Timer feature by specifying a Keep Alive Timeout value in:
• the KATO field of a Fabric Connect command (refer to section 6.3); or
• the KATO field of a Set Features command specifying the Keep Alive Timer feature (refer to section
5.1.25.1.8).
If an NVMe Transport binding specification requires the use of the Keep Alive Timer feature, and a
command attempts to disable the Keep Alive Timer by clearing the Keep Alive Timeout value to 0h, then
the controller shall abort that command with a status code of Keep Alive Timeout Invalid and the Keep Alive
Timeout value at the controller shall not be changed. If a command attempts to set the Keep Alive Timeout
value to a value that exceeds the maximum allowed by the associated NVMe Transport binding
specification, then the controller shall abort that command with a status code of Keep Alive Timeout Invalid
and the Keep Alive Timeout value at the controller shall not be changed. If a command sets the Keep Alive
Timeout value to a non-zero value that is less than the minimum supported by the NVMe Transport or less
than the minimum supported by the specific implementation, then the controller sets the Keep Alive Timeout
value to that minimum value.
As described in section 5.1.25.1.8, the controller rounds any Keep Alive Timeout value set by the host up
to the nearest granularity as reported in the Keep Alive Support (KAS) field (refer to Figure 312). To retrieve
the Keep Alive Timeout value being used by the controller, a host may issue a Get Features command for
the Keep Alive Timer feature.
 Keep Alive Timer Activation
The Keep Alive Timer is active if:
• CC.EN is set to ‘1’;
• CSTS.RDY is set to ‘1’;
• CC.SHN is cleared to ‘00b’;
• CSTS.SHST is cleared to ‘00b’; and
• the Keep Alive Timer feature is enabled as a result of the KATO field being set to a non-zero value
(refer to section 3.9.1).
Otherwise, the Keep Alive Timer is inactive, and a Keep Alive Timeout as described in sections 3.9.3.1 and
3.9.4.1 shall not occur  on the controller . Activating an inactive Keep Alive Timer (e.g., a Set Features
command successfully setting the Keep Alive Timeout value to a non-zero value from a value of 0h, or the
host enabling a controller that supports NVMe over Fabrics  where the Connect command specified a non-
zero Keep Alive Timeout value (refer to Figure 545)) shall initialize the Keep Alive Timer to the Keep Alive
Timeout value.
While the Keep Alive Timer is active, the host should ensure that the Admin Submission Queue has space
for a Keep Alive command.
 Command Based Keep Alive
For Command Based Keep Alive, the Keep Alive command is sent periodically from the host to the controller
on the Admin Queue. The completion of the Keep Alive command indicates that the host and controller are
able to communicate. For message -based transports, the Keep Alive Timeout is the maximum time an
association remains established without processing a Keep Alive command.
 Command Based Keep Alive on the Controller
The controller is using Command Based Keep Alive if the controller supports the Keep Alive Timer feature
and the TBKAS bit is cleared to ‘0’ in the Controller Attributes field in the Identify Controller data structure
(refer to Figure 312).
