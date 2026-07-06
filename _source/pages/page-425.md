---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 425
headings: ["5.1.25.1.31 Namespace Write Protection Config (Feature Identifier 84h)"]
---

# Source page 425

NVM Express® Base Specification, Revision 2.1

403
namespaces that are attached to the controller and that support PTPL (i.e., support reservations).  A Set
Features command that uses a valid namespace ID other than FFFFFFFFh and corresponds to a
namespace that supports reservations, modifies the PTPL state for that namespace. A Get Features
command that uses a namespace ID of FFFFFFFFh shall be aborted with a status code of Invalid Field in
Command. A Get Features command that uses a valid namespace ID other than FFFFFFFFh and
corresponds to a namespace that supports PTPL, returns the PTPL state for that namespace. If a Set
Features command or a Get Features command using a namespace ID other than FFFFFFFFh attempts
to access the PTPL state for a namespace that does not support this Feature Identifier, then the command
is aborted with status code of Invalid Field in Command.
This feature shall not be saveable (refer to Figure 195).
If a Get Features command successfully completes for this Feature Identifier, the attributes specified in
Figure 440 are returned in Dword 0 of the completion queue entry for that command
Figure 440: Reservation Persistence Configuration – Command Dword 11
Bits Description
31:01 Reserved
00
Persist Through Power Loss (PTPL):  If this bit is set to '1', then reservations and registrants persist
across a power loss. If this bit is cleared to ‘0’, then reservations are released and registrants are cleared
on a power loss.
5.1.25.1.31 Namespace Write Protection Config (Feature Identifier 84h)
This Feature is used by the host to configure the namespace write protection state or to determine the write
protection state of a namespace . Refer to section 8.1.16 for definition and behaviors of the namespace
write protection states. The settings are specified in Command Dword 11.
This Feature is not sav eable (refer to Figure 195). There is no default value for this Feature; the value of
the Feature after a power cycle or a Controller Level Reset is determined by the write protection state of
the namespace prior to the power cycle or Controller Level Reset, except for the Write Protect Until Power
Cycle write protection state (refer to section 8.1.16).
If a Get Features command is submitted for this Feature, the attributes specified in Figure 441 are returned
in Dword 0 of the completion queue entry for that command.
Figure 441: Write Protection – Command Dword 11
Bits Description
31:03 Reserved
02:00
Write Protection State (WPS): This field specifies the write protection state of the specified namespace.
Value Definition
000b No Write Protect
001b Write Protect
010b Write Protect Until Power Cycle
011b Permanent Write Protect
100b to 111b Reserved

If a Set Features command attempts to change the namespace write protection state of a namespace that
is in the Write Protect Until Power Cycle state or in the Permanent Write Protect state, then the controller
shall abort the command with a status code of Feature Not Changeable.
If a Set Features command attempts to change the namespace write protection state of a namespace to
the Write Protect Until Power Cycle state and the Write Protect Until Power Cycle Control (WPUPCC) bit
of the of the Write Protection Control field is cleared to ‘0’, then the controller shall abort the command with
a status code of Feature Not Changeable.
If a Set Features command attempts to change the namespace write protection state of a namespace to
the Write Protect Until Power Cycle state in a multi -domain NVM subsystem (i.e., the MDS bit is set to ‘1’
