---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 451
headings: []
---

# Source page 451

NVM Express® Base Specification, Revision 2.1

429
• only contain NVM subsystem basic discovery information entries or NVM subsystem extended
discovery information entries if the ETYPE field is set to 2h (i.e., a DDC is performing the register,
de-register, or update task). The DDC may set the Port Local (PO RTLCL) field to 1h if the NVM
subsystem discovery information entries being registered, de -registered, or updated are only for
NVM subsystem ports that are presented through the same NVM subsystem port on the DDC that
is performing the register, de-register, or update task.
Host extended discovery information entries and NVM subsystem extended discovery information entries
each contain the same set of fields, but not all of the fields are used for both extended discovery information
entry types. Refer to Figure 500 for the usage of the extended discovery information entry fields for each
extended discovery information entry type. If the entity performing a register, de -register, or update task
uses any field in an extended discovery information entry that conflicts with the value set in the ETYPE field
of the Discovery Information Management command data portion’s header (e.g., a host uses a field
intended for only NVM subsystem extended discovery information entries), then the controller shall abort
the command with a status code of Invalid Discovery Information.
Host extended discovery information entries shall contain at least one extended attribute containing a Host
Identifier. NVM subsystem extended discovery information entries may contain zero or more extended
attributes. The format for an extended attribute is defined in Figure 499.
Figure 495: Discovery Information Management – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 496: Discovery Information Management – Command Dword 10
Bits Description
31:04 Reserved
03:00
Task (TAS): This field selects the type of management task to perform.
Value Definition
0h Register
1h De-Register
2h Update
3h to Fh Reserved


Figure 497: Discovery Information Management – Data
Bytes O/M1 Description
Header
03:00 M Total Data Length (TDL):  This field specifies the length in bytes of the entire data
portion of the command.
07:04  Reserved
15:08 M
Number of Entries (NUMENT):  This field specifies the number of discovery
information entries being registered, de -registered, or updated. For a register or de -
register task, this field shall be non -zero. For an update task, this field shall be set to
2h.
