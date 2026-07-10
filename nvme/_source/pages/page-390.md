---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 390
headings: ["5.1.22.1 Command Completion", "5.1.23 Security Receive command"]
---

# Source page 390

NVM Express® Base Specification, Revision 2.1

368
Figure 372: Sanitize – Command Dword 10
Bits Description
02:00
Sanitize Action (SANACT): This field specifies the sanitize action to perform.
Value Definition
000b Reserved
001b Exit Failure Mode
010b Start a Block Erase sanitize operation
011b Start an Overwrite sanitize operation
100b Start a Crypto Erase sanitize operation
101b Exit Media Verification State
110b to 111b Reserved


Figure 373: Sanitize – Command Dword 11
Bits Description
31:00
Overwrite Pattern (OVRPAT): This field specifies a 32-bit pattern that is used for the Overwrite sanitize
operation. Refer to section 8.1.24.
If the Sanitize Action field is set to a value other than 011b (i.e., Overwrite), then this field shall be ignored
by the controller.
 Command Completion
When the command is complete, the controller shall post a completion queue entry to the Admin Completion
Queue indicating the status for the command. All sanitize operations are performed in the background (i.e.,
completion of the Sanitize command that started that sanitize operation does not indicate completion of the
sanitize operation). If a sanitize operation starts (refer to section 8.1.24.3), then the Sanitize Status log page
shall be updated before posting the completion queue entry for the command that started that sanitize
operation.
Sanitize command specific status values (i.e., SCT field set to 1h) are shown in Figure 374.
Figure 374: Sanitize – Command Specific Status Values
Value Definition
0Bh
Firmware Activation Requires Conventional Reset: The sanitize operation could not be started
because a firmware activation is pending and a Conventional Reset (refer to the NVM Express NVMe
over PCIe Transport Specification) is required.
10h Firmware Activation Requires NVM Subsystem Reset:  The sanitize operation could not be started
because a firmware activation is pending and an NVM Subsystem Reset is required.
11h Firmware Activation Requires Controller Level Reset: The sanitize operation could not be started
because a firmware activation is pending and a Controller Level Reset is required.
23h Sanitize Prohibited While Persistent Memory Region is Enabled: A sanitize operation is prohibited
while the Persistent Memory Region is enabled.
39h Controller Suspended: One or more controllers in the NVM subsystem have been suspended by a
Migration Send command.
 Security Receive command
The Security Receive command transfers the status and data result of one or more Security Send
commands that were previously submitted to the controller.
The association between a Security Receive command and previous Security Send commands is
dependent on the Security Protocol. The format of the data to be transferred is dependent on the Security
Protocol. Refer to SPC-5 for Security Protocol details.
Each Security Receive command returns the appropriate data corresponding to a Security Send command
as defined by the rules of the Security Protocol. The Security Receive command data may not be retained
if there is a loss of communication between the controller and host, or if a Controller Level Reset occurs.
