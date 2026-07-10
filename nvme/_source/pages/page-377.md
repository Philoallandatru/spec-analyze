---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 377
headings: ["5.1.17.1.2 Resume (Management Operation 1h)", "5.1.17.1.3 Set Controller State (Management Operation 2h)"]
---

# Source page 377

NVM Express® Base Specification, Revision 2.1

355
If a controller is suspended, then:
• accesses to controller properties for that suspended controller shall behave normally except a host
modification of the CC property that sets CC.EN bit from ‘0’ to ‘1’ (i.e., enabling that suspended
controller) has implementation specific behavior (i.e., the suspended controller may or may not set
the CSTS.RDY bit). A host should avoid modifications to controller properties, CMB, and PMR to a
controller that is suspended;
• the Controller Memory Buffer (CMB), if supported, shall behave as defined in section  8.2.1 (i.e.,
CMB on a suspended controller continues to function normally); and
• the Persistent Memory Region (PMR), if supported, shall behave as defined in section  8.2.4 (i.e.,
PMR on a suspended controller continues to function normally).
5.1.17.1.2 Resume (Management Operation 1h)
The Resume management operation of the Migration Send command is a request for the controller
processing the command to resume the operation of a different controller that is suspended (refer to section
5.1.17.1.1).
The Resume management operation uses the Command Dword 11 field as defined in Figure 352 and does
not use the Management Operation Specific field.
Figure 352: Resume – Command Dword 11
Bits Description
31:16 Reserved
15:00 Controller Identifier (CNTLID): This field specifies the identifier of the controller to resume operations.
The Resume management operation shall cause the controller specified by the Controller Identifier field
(refer to Figure 351) to start fetching and processing commands.
If the specified controller is not suspended (refer to section  5.1.17.1.1), then the controller shall abort the
command with a status code of Controller Not Suspended.
If the controller processing the command has received controller state for the specified controller to be
resumed and that controller state has not been verified and committed (refer to section  5.1.17.1.3), then
the controller shall abort the command with a status code of Command Sequence Error.
5.1.17.1.3 Set Controller State (Management Operation 2h)
The Set Controller State management operation of the Migration Send command allows the host to set the
state of the controller specified by the Controller Identifier ( CNTLID) field in Command Dword 11 (refer to
Figure 354).
The data buffer contains the Controller State data structure specified in Figure 358 which specifies the state
to be set in the specified controller.
To set the controller state of the specified controller, the specified controller is required to be in one or more
of these conditions:
• Suspended (refer to section 5.1.17.1.1);
• enabled (i.e., CC.EN bit is set to ‘1’); or
• offline, if that specified controller is a secondary controller.
If the specified controller is not in one of these conditions, then the controller processing the command shall
abort the command with a status code of Invalid Controller Identifier.
The host may submit one or more Migration Send commands to transfer the controller state. If more than
one Migration Send command is submitted to transfer the controller state, then:
• The host first submits a Migration Send command with:
o the Select field set to Set Controller State (i.e., 2h);
