---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 408
headings: []
---

# Source page 408

NVM Express® Base Specification, Revision 2.1

386
Figure 407: Host Behavior Support – Data Structure
Bytes Description
00
Advanced Command Retry Enable (ACRE): If this bit is set to 1h, then the Command Interrupted status
code is enabled (refer to Figure 102) and command retry delays are enabled. The controller may use the
Command Interrupted status code and may indicate a command retry delay by setting the Command
Retry Delay (CRD) field to a non-zero value in the Status field of a completion queue entry, refer to Figure
100. A host that sets this field to 1h indicates host support for the command retry behaviors that are
specified for both the Command Interrupted status code and non-zero values in the CRD field.
If this bit is cleared to 0h, then both the Command Interrupted status code and command retry delays
are disabled. The controller shall not use the Command Interrupted status code, and shall clear the CRD
field to 0h in all CQEs.
All values other than 0h and 1h are reserved.
01
Extended Telemetry Data Area 4 Supported (ETDAS): If this bit is set to 1h, then Telemetry Host -
Initiated Data Area 4 and Telemetry Controller -Initiated Data Area 4 are supported by the host. If the
Data Area 4 Support (DA4S) bit is set to ‘1’ of the Log Page Attributes field, then the controller may
populate Telemetry Host-Initiated Data Area 4 (refer to section 5.1.12.1.8) and the Telemetry Controller-
Initiated Data Area 4 (refer to section 5.1.12.1.9).
If this field is cleared to 0h, then Telemetry Host-Initiated Data Area 4 and Telemetry Controller-Initiated
Data Area 4 are not supported by the host.
All values other than 0h and 1h are reserved.
02
LBA Format Extension Enable (LBAFEE): I/O Command Set specific definition. Refer to the applicable
I/O Command Set specification for details.
All values other than 0h and 1h are reserved.
03
Host Dispersed Namespace Support (HDISNS): If this bit is set to 1h, then dispersed namespaces are
enabled. A host that sets this field to 1h specifies that globally unique namespace identifiers (refer to
section 8.1.9.2) are used for identifying namespaces (e.g., multi-path I/O software does not use the NSID
of a namespace as the sole method for detection of multiple paths to that namespace).
A host that supports reservations (refer to section 8.1.22) and sets this field to 1h specifies a Host
Identifier that is unique across all hosts that connect to any participating NVM subsystem.
If this bit is cleared to 0h, then dispersed namespaces are disabled. If the participating NVM subsystem
prohibits host access to dispersed namespaces (refer to section 8.1.9.4) when this field is cleared to 0h,
then the controller aborts any I/O commands (refer to section 7 in this specification and the I/O
Commands section in the appropriate I/O Command Set specification) or any of the Admin commands
listed in Figure 615 that the host submits to dispersed namespaces with a status code of Host Dispersed
Namespace Support Not Enabled, as described in section 8.1.9.4.
All values other than 0h and 1h are reserved.
