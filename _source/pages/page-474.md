---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 474
headings: []
---

# Source page 474

NVM Express® Base Specification, Revision 2.1

452
• If the host specifies a Controller ID value of FFFFh for the Admin Queue, then the controller shall
abort the command with a status code of Connect Invalid Parameters.
The NVM subsystem may allocate specific controllers to particular hosts. If a host requests a controller that
is not allocated to that host, then  the controller shall abort the command with  a status code of Connect
Invalid Host. The mechanism for allocating specific controllers to particular hosts is outside the scope of
this specification.
The host shall establish an association with a controller and enable the controller before establishing a
connection with an I/O Queue of the controller. If the host sends a Connect command specifying a Queue
ID for an Admin Queue or I/O Queue that has already been created, then the controller shall abort the
command with a status code of Command Sequence Error.
The Controller shall abort a Connect command with a status code of Connect Invalid Parameters if:
• the host sends a Connect command to create an I/O Queue while the controller is disabled;
• the Host NQN, NVM Subsystem NQN, and the Controller ID values specified for an I/O Queue are
not the same as the values specified for the associated Admin Queue in which the association
between the host and controller was established;
• the Host Identifier for an I/O Queue is not set to a value of 0h and is not set to the same value as
the value specified for the associated Admin Queue in which the association between the host and
controller was established;
• the Host NQN or NVM Subsystem NQN values do not match the values that the NVM subsystem
is configured to support;
• there is a syntax error in the Host NQN or NVM Subsystem NQN value (refer to section 4.7); or
• the host specifies a Controller ID value in the range FFF0h to FFFDh.
If the NVMe Subsystem Port, NVMe Transport Type or NVMe Transport Address used by the NVMe
Transport (refer to section  6.3) are not the same as the values used for the associated Admin Queue in
which the association between the host and controller was established, then it is possible that the Connect
command is not received by an NVM subsystem. If the Connect command is recei ved by an NVM
subsystem, then:
• the NVM subsystem that receives the command may not be the same NVM subsystem to which
the association between the host and controller was established (i.e., the NVMe Transport Type
and NVMe Transport Address are unique to an NVM Subsystem Port); and
• the values of the NVM Subsystem NQN or Controller ID may not be valid at that NVM Subsystem
Port (e.g., the NVM Subsystem NQN may specify a different NVM subsystem than the one that
received that Connect command, or the Controller ID may specify a controller that is already bound
to a different NVM Subsystem Port).
If this situation occurs and the Connect command  is aborted, then the status code shall be set to Connect
Invalid Parameters. There is no requirement that such a Connect command be received by an NVM
subsystem (e.g., if the NVMe Transport Address is not a valid transport address, or is the address of a
fabric endpoint that does not support NVMe over Fa brics, then the resulting error, if any, is specific to the
fabric).
Submission Queue (SQ) flow control based on the SQ Head Pointer (SQHD) field in Fabrics response
capsules (refer to section 3.3.2.1.2) shall be supported by all hosts and controllers. Use of SQ flow control
is negotiated by the Connect command and response. A host requests that SQ flow control be disabled by
setting the Disable SQ Flow Control (DISSQFC) bit  of the Connect Attributes field to ‘1’ in a Connect
command. A controller that agrees to disable SQ flow control shall set the SQHD field to FFFFh in the
response to that Connect command. A controller that does not agree to disable SQ flow control shall s et
the SQHD field to a value other than FFFFh in the response to that Connect command.
If the Connect command did not request that SQ flow control be disabled, then the controller shall not set
the SQHD field to FFFFh in the response to that Connect command.
SQ flow control is disabled and shall not be used for a created queue pair only if:
a) the DISSQFC bit is set to ‘1’ in the Connect command that creates the queue pair; and
