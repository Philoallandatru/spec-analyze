---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 373
headings: ["5.1.16.1 Migration Receive Management Operations", "5.1.16.1.1 Get Controller State (Management Operation 0h)"]
---

# Source page 373

NVM Express® Base Specification, Revision 2.1

351
Figure 343: Migration Receive – Command Dword 14
Bits Description
31:07 Reserved
06:00 UUID Index (UIDX): Refer to Figure 658.

Figure 344: Migration Receive – Command Dword 15
Bits Description
31:00
Number of Dwords (NUMDL):  This field specifies the number of dwords to return. If the host specifies a
value that is greater than the number of dwords returned by the specified management operation of the
command, then the controller returns the number of dwords defined by the specified management operation
of the command with undefined results for the remaining dwords. This is a 0’s based value.
 Migration Receive Management Operations
5.1.16.1.1 Get Controller State (Management Operation 0h)
The Get Controller State management operation of the Migration Receive command allows the host to
retrieve from the controller processing the command state data for the controller specified in the Controller
Identifier field in the Management Operation Spe cific field (refer to  Figure 345). The data returned in the
data buffer is the Controller State data structure as defined by Figure 358.
If the controller specified in the Controller Identifier field in the Management Operation Specific field (i.e.,
the specified controller):
• is suspended (refer to section 5.1.17.1.1); and
• remains suspended during the processing of the Migration Receive command,
then:
• the specified controller has stopped processing commands; and
• if:
o a Controller Level Reset has not occurred on the specified controller during the processing
of the Migration Receive command; or
o the host does not modify controller properties on the specified controller during the
processing of the Migration Receive command,
then the returned Controller State data structure is consistent between each field reported in that
data structure.
If the specified controller:
• is not suspended during the processing of the Migration Receive command; or
• has a Controller Level Reset during the processing of the Migration Receive command,
then the specified controller state may be changing during the processing of the Migration Receive
command and:
• each field in the reported Controller State data structure contains the value that reflects the state
of the specified controller at the time the value was obtained by the controller processing the
Migration Receive command; and
• state of the specified controller may be at a different state when a value is obtained between
different fields (i.e., the returned Controller State data structure may or may not be internally
consistent between each field).
The Get Controller State management operation uses the Management Operation Specific field as defined
in Figure 345 and the Command Dword 11 field as defined in Figure 346.
