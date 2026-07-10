---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 473
headings: ["6.3 Connect Command and Response"]
---

# Source page 473

NVM Express® Base Specification, Revision 2.1

451
Figure 543: Authentication Send Command – Submission Queue Entry
Bytes Description
47:44 Transfer Length (TL): The value of this field is specific to the Security Protocol as defined in SPC-5
where INC_512 is cleared to ‘0’.
63:48 Reserved

Figure 544: Authentication Send Response
Bytes Description
07:00 Reserved
09:08 SQ Head Pointer (SQHD): Indicates the current Submission Queue Head pointer for the associated
Submission Queue.
11:10 Reserved
13:12 Command Identifier (CID): Indicates the identifier of the command that is being completed.
15:14
Status Info (STS): Indicates status for the command.
Bits Description
15:01 Status (STATUS): The Status field for the command. Refer to section 4.2.3.
00 Reserved

6.3 Connect Command and Response
The Connect command is used to create a Submission and Completion Queue pair. If the Admin Queue is
specified, then the Connect command establishes an association between a host and a controller. The
fields for the submission queue entry are defined in Figure 545 and the fields for the data portion are defined
in Figure 546.
A host that uses a single Host NQN may employ multiple Host Identifiers to designate elements of the host
that access an NVM subsystem independently of each other (e.g., physical or logical partitions of the host).
Alternatively, a host may employ multiple  Host NQN values to cause each element to be treated as a
separate host by an NVM subsystem.
If an NVM subsystem supports DH-HMAC-CHAP authentication (refer to section 8.3.4), then the Host NQN
and the NVM Subsystem NQN parameters in a Connect command are required to be different. If the Host
NQN and the NVM Subsystem NQN parameters in a Connect command are identical and the NVM
subsystem supports DH-HMAC-CHAP authentication, then the controller shall abort the command with  a
status code of Connect Invalid Host.
The NVM subsystem shall not allocate a Controller ID in the range FFF0h to FFFFh as a valid Controller
ID on completion of a Connect command. Controller IDs FFF0h to FFFFh are defined in Figure 27. If the
host is not allowed to establish an association to any controller in the NVM subsystem, then the controller
shall abort the command with a status code of Connect Invalid Host.
If the NVM subsystem supports the dynamic controller model, then:
• the Controller ID of FFFFh is specified as the Controller ID in a Connect command for the Admin
Queue. If the controller ID is not set to FFFFh, then  the controller shall abort the command with  a
status code of Connect Invalid Parameters;
• the NVM subsystem shall allocate any available controller to the host; and
• return that allocated Controller ID in the Connect response.
If the NVM subsystem supports the static controller model, then:
• The host is able to request a specific controller in a Connect command for the Admin Queue. If the
host is not allowed to establish an association to the specified controller, then the controller shall
abort the command with a status code of Connect Invalid Host;
• The Controller ID of FFFEh on the Admin Queue specifies that any Controller ID may be allocated
and returned in the Connect response; and
