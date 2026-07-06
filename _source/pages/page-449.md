---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 449
headings: ["5.3.2.1 Command Completion", "5.3.3 Discovery Information Management command"]
---

# Source page 449

NVM Express® Base Specification, Revision 2.1

427
Figure 493: Create Exported NVM Subsystem – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 494: Create Exported NVM Subsystem – Command Dword 10
Bits Description
31:09 Reserved
08
Restricted Access (RA): A value of ‘0’ configures this Exported NVM Subsystem for unrestricted access
and may be accessed by any host. A value of ‘1’ configures this Exported NVM Subsystem to restrict access
to Host NQNs present in this Exported NVM Subsystem’s Allowed Host List.
07:00 Reserved
The Restricted Access bit defined in Figure 494 specifies the initial access setting for the Exported NVM
Subsystem.
 Command Completion
Upon successful completion of the Create Exported NVM Subsystem command:
• a new Exported NVM Subsystem is created with the Access Control mode specified in the
Restricted Access bit of the command:
a) the newly created Exported NVM Subsystem shall have an empty list of allowed Host NQNs;
and
b) there are no Exported Namespaces linked to the Exported NVM Subsystem;
• the controller posts a completion queue entry to the Admin Completion Queue indicating the status
for the command (refer to section 8.3.3 for usages); and
• the SUBNQN for the newly created Exported NVM Subsystem shall be returned in the command
data buffer.
 Discovery Information Management command
The Discovery Information Management command registers, de-registers, or updates discovery information
entries.
The type of task performed by the Discovery Information Management command (i.e., registration, de -
registration, or update) is determined by the value set in Task (TAS) field of Command Dword 10 (refer to
Figure 496).
For a register task (i.e., TAS field cleared to 0h), the Discovery Information Management command
registers:
• one or more host discovery information entries; or
• one or more NVM subsystem discovery information entries.
For a de -register task (i.e., TAS field set to 1h), the Discovery Information Management command de -
registers:
• one or more host discovery information entries; or
• one or more NVM subsystem discovery information entries.
For an update task (i.e., TAS field set to 2h), the Discovery Information Management command updates:
• one host discovery information entry; or
• one NVM subsystem discovery information entry.
Discovery information entries may be one of the following:
