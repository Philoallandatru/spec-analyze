---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 450
headings: []
---

# Source page 450

NVM Express® Base Specification, Revision 2.1

428
• a basic discovery information entry (refer to Figure 294); or
• an extended discovery information entry (refer to Figure 498).
Host discovery information entries shall be extended discovery information entries.
NVM subsystem discovery information entries may be basic discovery information entries or may be
extended discovery information entries. Each NVM subsystem discovery information entry may specify an
NVM subsystem that exposes namespaces that hosts may acce ss or may specify a referral to another
Discovery subsystem.
The Discovery Information Management command uses the Data Pointer field as shown in Figure 495 and
Command Dword 10 as shown in Figure 496. All other command specific fields are reserved.
The data portion of the Discovery Information Management command contains a header that identifies the
entity performing the register, de-register, or update task.
If a register task is being performed and the data portion of the Discovery Information Management
command does not contain one or more discovery information entries, then the controller shall abort the
command with status code of Invalid Field in Command.  If the number of discovery information entries
contained in the data portion of the Discovery Information Management command exceeds the available
capacity for new discovery information entries on the CDC or DDC, then the controller shall abort the
command with a status code of Insufficient Discovery Resources. If multiple register tasks are performed
by the same entity (i.e., the value set in the Entity Identifier (EID) field of the header is associated with an
existing registration record contained in the CDC or DDC), and:
a) the Entry Key associated with a discovery information entry in the Discovery Information
Management command matches the Entry Key associated with an existing registration record, then
the existing registration record contained in the CDC or DDC shall be up dated with the discovery
information entry from the Discovery Information Management command; or
b) the Entry Key associated with a discovery information entry in the Discovery Information
Management command does not match the Entry Key associated with an existing registration
record, then the discovery information entry in the Discovery Information Mana gement command
shall be registered with the CDC or DDC.
If a de -register task is being performed and the data portion of the Discovery Information Management
command does not contain one or more discovery information entries, then the controller shall abort the
command with a status code of Invalid Field in Command.
If an update task is being performed and the data portion of the Discovery Information Management
command does not contain two discovery information entries, then the controller shall abort the command
with a status code of Invalid Field in Command. The fi rst discovery information entry identifies the
registration record contained in the CDC or DDC that is to be updated. The fields in the first discovery
information entry that are not used as part of the Entry Key are ignored. The second discovery informati on
entry replaces the existing registration record identified by the Entry Key from the first discovery information
entry. The update task shall be atomic.
The format for the data portion of the Discovery Information Management command is defined in  Figure
497.
Based upon the value set in the Entity Type (ETYPE) field of the header, the data portion of a Discovery
Information Management command shall:
• only contain host extended discovery information entries if the ETYPE field is set to 1h (i.e., a host
is performing the register, de-register, or update task);
• only contain host extended discovery information entries if the ETYPE field is set to 3h (i.e., a CDC
is performing the register, de-register, or update task); and
