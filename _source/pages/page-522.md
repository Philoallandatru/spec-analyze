---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 522
headings: ["8.1.8.2 Identify (Directive Type 00h)", "8.1.8.2.1 Directive Receive", "8.1.8.2.1.1 Return Parameters (Directive Operation 01h)"]
---

# Source page 522

NVM Express® Base Specification, Revision 2.1

500
• if one or more I/O Command Directives is enabled and the DTYPE field is set to a value that is not
supported or not enabled, then the controller shall abort the command with a status code of Invalid
Field in Command.
For the Streams Directive (i.e., DTYPE field set to 01h), if the DSPEC field is cleared to 0h in a n I/O
command that supports the Streams Directive, then that I/O command shall be processed normally (i.e., as
if DTYPE field is cleared to 00h).
 Identify (Directive Type 00h)
The Identify Directive is used to determine the Directive Types that the controller supports and to enable
use of the supported Directives. If Directives are supported, then this Directive Type shall be supported.
The Directive operations that shall be supported for the Identify Directive are listed in Figure 599.
Figure 599: Identify Directive – Directive Operations
Directive Command Directive Operation Name Directive Operation Value Reference
Directive Receive Return Parameters 01h 8.1.8.2.1.1
Reserved All other values
Directive Send Enable Directive 01h 8.1.8.2.2.1
Reserved All other values
8.1.8.2.1 Directive Receive
This section defines operations used with the Directive Receive command for the Identify Directive.
 Return Parameters (Directive Operation 01h)
This operation returns a data structure that contains a bit vector specifying the Directive Types supported
by the controller and a bit vector specifying the Directive Types enabled for the namespace. The data
structure returned is defined in Figure 600. If an NSID value of FFFFFFFFh is specified, then the controller
shall abort the command with a status code of Invalid Field in Command. The DSPEC field in command
Dword 11 is not used for this operation.
Figure 600: Identify Directive – Return Parameters Data Structure
Bytes Bits Description
 Directives Supported
31:00
255:16 Reserved
15 Vendor Specific Directive (VSDIRS): This bit is set to ‘1’ if the Vendor Specific Directive
is supported. This bit is cleared to ‘0’ if the Vendor Specific Directive is not supported.
14:03 Reserved
02 Data Placement Directive (DPDIRS): This bit is set to ‘1’ if the Data Placement Directive
is supported. This bit is cleared to ‘0’ if the Data Placement Directive is not supported.
01 Streams Directive (SDIRS): This bit is set to ‘1’ if the Streams Directive is supported. This
bit is cleared to ‘0’ if the Streams Directive is not supported.
00 Identify Directive (IDIRS): This bit shall be set to ‘1’ to indicate that the Identify Directive
is supported.
 Directives Enabled
63:32
255:16 Reserved
15 Vendor Specific Directive (VSDIRE): This bit is set to ‘1’ if the Vendor Specific Directive
is enabled. This bit is cleared to ‘0’ if the Vendor Specific Directive is not enabled.
14:03 Reserved
02 Data Placement Directive (DPDIRE): This bit is set to ‘1’ if the Data Placement Directive
is enabled. This bit is cleared to ‘0’ if the Data Placement Directive is not enabled.
01 Streams Directive (SDIRE): This bit is set to ‘1’ if the Streams Directive is enabled. This
bit is cleared to ‘0’ if the Streams Directive is not enabled.
00 Identify Directive (IDIRE): This bit shall be set to ‘1’ to indicate that the Identify Directive
is enabled.
 Directive Persistent Across Controller Level Resets
