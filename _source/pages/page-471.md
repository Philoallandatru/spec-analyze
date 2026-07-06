---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 471
headings: ["6 Fabrics Command Set", "6.1 Authentication Receive Command and Response"]
---

# Source page 471

NVM Express® Base Specification, Revision 2.1

449
6 Fabrics Command Set
Fabrics commands are used to create queues and initialize a controller. Fabrics commands have an
Opcode field of 7Fh and are distinguished by the Fabrics Command Type as shown in Figure 540. Fabrics
commands are processed regardless of the state of controller enable (CC.EN). The Fabrics command
capsule is defined in section 3.3.2.1.1 and the Fabrics response capsule and status is defined in section
3.3.2.1.2. The common Fabrics Submission Queue entry is shown in Figure 94 and the common Fabrics
Completion Queue entry is shown in Figure 99.
Restrictions on processing commands listed in Figure 540 are defined in the Admin Command Set in
section 5 (e.g., while the NVM subsystem is performing a sanitize operation or processing of a Format NVM
command).
Figure 540: Fabrics Command Type
Some Fabric Command Type by
Field
Combined
Fabrics
Command
Type2
O/M1 I/O Queue3 Command (07:02)  (01:00)
Function Data Transfer4
0000 00b 00b 00h M No Property Set
0000 00b 01b 01h M Yes Connect5
0000 01b 00b 04h M No Property Get
0000 01b 01b 05h O Yes Authentication Send
0000 01b 10b 06h O Yes Authentication Receive
0000 10b 00b 08h O Yes Disconnect
Vendor Specific
11xx xxb Note 4 C0h to FFh O  Vendor specific
Notes:
3. O/M definition: O = Optional, M = Mandatory.
4. Fabrics Command Types not listed are reserved.
5. All Fabrics commands, other than the Disconnect command, may be submitted on the Admin Queue. The I/O Queue
supports Fabrics commands as specified in this column. If a Fabrics command that is not supported on an I/O Queue
is sent on an I/O Queue, that command shall be aborted with a status code of Invalid Field in Command.
6. 00b = no data transfer; 01b = host to controller; 10b = controller to host; 11b = reserved . Refer to the Transfer
Direction field in Figure 94.
7. The Connect command is submitted and completed on the same queue that the Connect command creates. Refer
to section 3.3.2.2.
6.1 Authentication Receive Command and Response
The Authentication Receive command transfers the status and data result of one or more Authentication
Send commands that were previously submitted to the controller.
The association between an Authentication Receive command and previous Authentication Send
commands is dependent on the Security Protocol. The format of the data to be transferred is dependent on
the Security Protocol. Refer to SPC-5 for Security Protocol details.
Authentication Receive commands return the appropriate data corresponding to an Authentication Send
command as defined by the rules of the Security Protocol. The Authentication Receive command data shall
not be retained if there is a loss of communication between the controller and host, or if a Controller Level
Reset occurs.
Figure 541: Authentication Receive Command – Submission Queue Entry
Bytes Description
03:00 Command Dword 0 (CDW0): Refer to Figure 95.
04 Fabrics Command Type (FCTYPE): Set to 06h to specify an Authentication Receive command.
23:05 Reserved
