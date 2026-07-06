---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 477
headings: []
---

# Source page 477

NVM Express® Base Specification, Revision 2.1

455
Figure 546: Connect Command – Data
Bytes Description
15:00
Host Identifier (HOSTID):  This field has the same definition as the Host Identifier defined in
section 5.1.25.1.28.
For a Connect command to create an Admin Queue (i.e., the QID field is cleared to 0h) that
completes successfully the controller shall set the current Host Identifier (refer to section
5.1.25.1.28.2) to this value.
For a Connect command to create an I/O queue (i.e., the QID field is set to a non-zero value):
• if this field is cleared to 0h, then the controller shall ignore this field; and
• if this field is set to a non-zero value and the value in this field does not match the value
in the Host Identifier feature, then the controller shall abort the command with a status
code of Connect Invalid Parameters.
17:16
Controller ID (CNTLID): Specifies the controller ID requested. This field corresponds to the
Controller ID (CNTLID) value returned in the Identify Controller data structure for a particular
controller. If the NVM subsystem uses the dynamic controller model, then the value shall be
FFFFh for the Admin Queue and any available controller may be returned. If the NVM subsystem
uses the static controller model and the value is FFFEh for the Admin Queue, then any available
controller may be returned.
255:18 Reserved
511:256 NVM Subsystem NVMe Qualified Name (SUBNQN): NVMe Qualified Name (NQN) that
uniquely identifies the NVM subsystem. Refer to section 4.7.
767:512 Host NVMe Qualified Name (HOSTNQN): NVMe Qualified Name (NQN) that uniquely
identifies the host. Refer to section 4.7.
1023:768 Reserved
The Connect response provides status for the Connect command. If a connection is established, then the
Controller ID allocated to the host is returned. The Connect response is defined in Figure 547.
For a Connect command that fails, the controller shall not:
• return a status code of Invalid Field in Command; and
• add an entry to the Error Information log page (refer to section 5.1.12.1.2).
Figure 547: Connect Response
Bytes Description
03:00 Status Code Specific (SCS): The value is dependent on the status returned. Refer to Figure 548.
07:04 Reserved
09:08
SQ Head Pointer (SQHD): If the Connect command requested that SQ flow control be disabled, then a
value of FFFFh in this field indicates that SQ flow control is disabled for the created queue pair.
Otherwise, this field indicates the current Submission Queue Head pointer for the associated
Submission Queue and also indicates that SQ flow control is enabled for the created queue pair.
11:10 Reserved
13:12 Command Identifier (CID): Indicates the identifier of the command that is being completed.
15:14
Status Info (STS): Indicates status for the command.
Bits Description
15:01 Status (STATUS): The Status field for the command. Refer to section 4.2.3. Refer
to Figure 105 for values specific to the Connect command.
00 Reserved
