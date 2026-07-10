---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 330
headings: []
---

# Source page 330

NVM Express® Base Specification, Revision 2.1

308
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description

11
Host Managed Live Migration Support (HMLMS):  If this bit is set to ‘1’,
then the controller supports the Host Managed Live Migration capability
(refer to section 8.1.12).
If this bit is cleared to ‘0’, then the controller does not support the Host
Managed Live Migration capability.
Secondary controllers shall clear this bit to ‘0’.
10
Command and Feature Lockdown Supported (CFLS): If this bit is set to
‘1’, then the controller supports the Command and Feature Lockdown
capability (refer to section 8.1.5). If this bit is cleared to ‘0’, then the controller
does not support the Command and Feature Lockdown capability. The value
of this bit shall be the same for all controllers in the NVM subsystem.
9
Get LBA Status Supported (GLSS): If this bit is set to ‘1’, then the controller
supports the Get LBA Status capability with the Action Type values of 10h
and 11h (refer to the NVM Command Set Specification). If this bit is cleared
to ‘0’, then the controller does not support the Get LBA Status capability with
the Action Type values of 10h and 11h.
8
Doorbell Buffer Config Supported (DBCS): If this bit is set to '1', then the
controller supports the Doorbell Buffer Config command. If this bit is cleared
to '0', then the controller does not support the Doorbell Buffer Config
command.
7
Virtualization Management Supported (VMS): If this bit is set to ‘1’, then
the controller supports the Virtualization Management command. If this bit is
cleared to ‘0’, then the controller does not support the Virtualization
Management command.
6
NVMe-MI Send Receive Supported (NSRS): If this bit is set to ‘1’, then the
controller supports the NVMe -MI Send and NVMe -MI Receive commands.
If this bit is cleared to ‘0’, then the controller does not support the NVMe -MI
Send and NVMe-MI Receive commands.
5
Directives Supported (DIRS):  If this bit is set to ‘1’, then the controller
supports Directives. If this bit is cleared to ‘0’, then the controller does not
support Directives. A controller that supports Directives shall support the
Directive Send and Directive Receive commands. Refer to section 8.1.8.
4
Device Self -test Supported (DSTS):  If this bit is set to ‘1’, then the
controller supports the Device Self-test command. If this bit is cleared to ‘0’,
then the controller does not support the Device Self-test command.
3
Namespace Management Supported (NMS):  If this bit is set to ‘1’, then
the controller supports the Namespace Management capability  (refer to
section 8.1.15). If this bit is cleared to ‘0’, then the controller does not support
the Namespace Management capability.
2
Firmware Download Supported (FWDS):  If this bit is set to ‘1’, then the
controller supports the Firmware Commit and Firmware Image Download
commands. If this bit is cleared to ‘0’, then the controller does not support
the Firmware Commit and Firmware Image Download commands.
1
Format NVM Supported (FNVMS): If this bit is set to ‘1’, then the controller
supports the Format NVM command. If this bit is cleared to ‘0’, then the
controller does not support the Format NVM command.
0
Security Send Receive Supported (SSRS): If this bit is set to ‘1’, then the
controller supports the Security Send and Security Receive commands. If
this bit is cleared to ‘0’, then the controller does not support the Security
Send and Security Receive commands.
258 M M R
Abort Command Limit (ACL): This field indicates the maximum number of concurrently
executing Abort commands (refer to section 5.1) on the Admin Queue supported by the
controller. This is a 0’s based value. It is recommended that implementations support
concurrent execution of a minimum of four Abort commands.
