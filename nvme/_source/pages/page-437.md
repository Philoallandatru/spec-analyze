---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 437
headings: ["5.1.27.1.2 Track Memory Changes (Management Operation 1h)"]
---

# Source page 437

NVM Express® Base Specification, Revision 2.1

415
Figure 464: Log User Data Changes – Management Operation Specific Field
Bits Description
15:04 Reserved
03:00
Logging Action (LACT): This field specifies the logging action to be performed.
Value Definition
0h
Stop Logging:  The controller shall stop logging user data changes to namespaces
attached to the controller associated with the User Data Migration Queue specified in the
Controller Data Queue Identifier field.
1h
Start Logging:  The controller shall start logging user data changes to namespaces
attached to the controller associated with the User Data Migration Queue  (refer to the
CDQID field in Figure 465) into that User Data Migration Queue where those user data
changes are cause d by the controller associated with that User Data Migration Queue
processing commands.
2h to Fh Reserved

The Log User Data Changes management operation uses Command Dword 11 as defined in Figure 465.
Figure 465: Log User Data Changes – Command Dword 11
Bits Description
31:16 Reserved
15:00 Controller Data Queue Identifier (CDQID): This field specifies the Controller Data Queue Identifier of the
User Data Migration Queue.
Refer to the applicable I/O Command Set specification for any requirements on the posting of entries into
the User Data Migration Queue if this command is successful.
If the value in the Controller Data Queue Identifier field specifies a User Data Migration Queue that does
not exist in the controller processing the command, then the controller shall abort the command with a
status code of Invalid Controller Data Queue.
If the LACT field is set to 1h and any controller in the NVM subsystem is already logging user data for the
controller associated with the Controller Data Queue specified by the CDQID field, then the controller shall
abort the command with a status code of Invalid Controller Data Queue.
If the controller associated with the User Data Migration Queue specified by the CDQID field is suspended
as a result of a Migration Send command (refer to section  5.1.17.1.1), then the controller shall abort the
command with a status code of Controller Suspended.
5.1.27.1.2 Track Memory Changes (Management Operation 1h)
The Track Memory Changes management operation of the Track Send command requests the controller
processing that Track Send command to:
• start tracking host memory changes described by the specified host memory ranges that are
caused by the processing commands in the controller specified by Controller Identifier field (refer
to Figure 467); or
• stop tracking the host memory changes that are caused by the processing commands by the
controller specified by Controller Identifier field.
If the TACT bit is set to ‘1’ (refer to Figure 466), then:
• The data buffer contains a Track Memory Changes data structure (refer to Figure 468) that
specifies one or more host memory ranges associated with the controller specified by the
Controller Identifier field.
• If the value in the Requested Memory Range Tracking Granularity (RMRTG) field  is greater than
the value in the Maximum Memory Range Tracking Granularity (MAXMRTG) field in the Identify
Controller data structure (refer to  Figure 312), then the controller shall abort the command with a
status code of Invalid Field in Command.
