---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 285
headings: []
---

# Source page 285

NVM Express® Base Specification, Revision 2.1

263
Figure 265: Command and Feature Lockdown Log Specific Parameter Field
Bits Description
14 Reserved
13:12
Contents (CNTTS ): This field in combination with the Scope field specifies the contents of the
Command and Feature Identifier List field in the log page.
Value Command and Feature Identifier List Definition
00b List of command opcodes or Feature Identifiers specified by the Scope field that are able
to be prohibited.
01b List of command opcodes or Feature Identifiers specified by the Scope field that are
currently prohibited if received on an Admin submission queue.
10b List of command opcodes or Feature Identifiers specified by the Scope field that are
currently prohibited if received out-of-band on a Management Endpoint.
11b Reserved

11:08
Scope (SCP): This field in combination with the Contents field specifies the contents of the
Command and Feature Identifier List field in the log page.
Value Command and Feature Identifier List Definition
0h List of Admin Command Set opcodes
1h Reserved
2h List of Feature Identifiers
3h List of Management Interface Command Set opcodes (refer to the NVM Express
Management Interface Specification)
4h List of PCIe Command Set opcodes (refer to the NVM Express Management Interface
Specification)
5h to Fh Reserved

If a UUID Index is specified in the Get Log Page command (refer to section 5.1.12) and the Scope field is
set to 2h, then the controller should return vendor specific Set Features lockdown information based on that
UUID index. If the Scope field is not set to 2h, then the controller ignores the UUID index field.
If a controller processes this command with the Contents field set to 10b and the NVM subsystem does not
contain a Management Endpoint, then the command shall be aborted with a status code of Invalid Field in
Command.
The log page returned is defined in Figure 266.
