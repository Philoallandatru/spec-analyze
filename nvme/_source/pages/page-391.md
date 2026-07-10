---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 391
headings: ["5.1.23.1 Command Completion", "5.1.23.2 Security Protocol 00h", "5.1.23.3 Security Protocol EAh", "5.1.24 Security Send command"]
---

# Source page 391

NVM Express® Base Specification, Revision 2.1

369
The fields used are Data Pointer, Command Dword 10, and Command Dword 11 fields. All other command
specific fields are reserved.
Figure 375: Security Receive – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 376: Security Receive – Command Dword 10
Bits Description
31:24
Security Protocol (SECP): This field specifies the security protocol as defined in SPC-5. The controller
shall abort the command with a status code of Invalid Field in Command if an unsupported value of the
Security Protocol is specified.
23:16 SP Specific 1 (SPSP1): The value of this field contains bits 15:08 of the Security Protocol Specific field
as defined in SPC-5.
15:08 SP Specific 0 (SPSP0): The value of this field contains bits 07:00 of the Security Protocol Specific field
as defined in SPC-5.
07:00 NVMe Security Specific Field (NSSF): Refer to Figure 378 for definition of this field for Security Protocol
EAh. For all other Security Protocols this field is reserved.

Figure 377: Security Receive – Command Dword 11
Bits Description
31:00 Allocation Length (AL): The value of this field is specific to the Security Protocol In command with the
INC_512 field cleared to 0h as defined in SPC-5.
 Command Completion
If the command is completed, then the controller shall post a completion queue entry to the Admin
Completion Queue indicating the status for the command.
 Security Protocol 00h
A Security Receive command with the Security Protocol field cleared to 00h shall return information about
the security protocols supported by the controller. This command is used in the security discovery process
and is not associated with a Security Send command. Refer to SPC-5 for the details of Security Protocol
00h and the SP Specific field.
 Security Protocol EAh
Security Protocol EAh is assigned for NVMe interface use (refer to ACS -4). This protocol may be used in
Security Receive and Security Send commands. The specific usage type is defined by the Security Protocol
Specific Field defined in Figure 378.
Figure 378: Security Protocol EAh – Security Protocol Specific Field Values
SP Specific (SPSP)
Value Definition NVMe Security Specific Field (NSSF)
Definition
0001h Replay Protected Memory Block RPMB Target
0002h to FFFFh Reserved Reserved
 Security Send command
The Security Send command is used to transfer security protocol data to the controller. The data structure
transferred to the controller as part of this command contains security protocol specific commands to be
performed by the controller. The data structure transferred may also contain data or parameters associated
with the security protocol commands. Status and data that is to be returned to the host for the security
