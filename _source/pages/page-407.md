---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 407
headings: ["5.1.25.1.14 Host Behavior Support (Feature Identifier 16h)"]
---

# Source page 407

NVM Express® Base Specification, Revision 2.1

385
The transition to the window selected is complete when the Set Features command completes successfully.
A transition to the Deterministic Window may be delayed (i.e., the Set Features command completion is
delayed) if the minimum time has not been spent in the Non-Deterministic Window.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 406 are returned
in Dword 0 of the completion queue entry for that command. If Predictable Latency Mode is not enabled,
then the controller shall abort the command with a status code of Invalid Field in Command.
Figure 405: Predictable Latency Mode Window – Command Dword 11
Bits Description
31:16 Reserved
15:00 NVM Set Identifier (NVMSETID): This field specifies the NVM Set to be modified.

Figure 406: Predictable Latency Mode Window – Command Dword 12
Bits Description
31:03 Reserved
02:00
Window Select (WSEL): This field selects or indicates the window used by all namespaces in the NVM
Set.
Value Definition
000b Reserved
001b Deterministic Window (DTWIN)
010b Non-Deterministic Window (NDWIN)
011b to 111b Reserved

5.1.25.1.14 Host Behavior Support (Feature Identifier 16h)
This Feature enables use of controller functionality that is associated with and depends upon specific host
behavior that may or may not be supported by all hosts. A controller does not use such functionality unless
the host has indicated that the host  supports the specific host behavior upon which the functionality
depends. The host indicates that support to the controller by setting a field in this Feature. That host action
enables controller use of the associated functionality with that host. A contr oller shall not use functionality
with a host that has not indicated support for the associated specific host behavior upon which that controller
functionality depends. The attributes in Figure 407 are transferred in the data buffer.
For example, the Command Interrupted status code is associated with and depends upon the specific host
behavior that the host is expected to retry commands that are aborted with that status code. That command
retry behavior may or may not be supported by all hosts (e.g., hosts compliant with versions 1.3 and earlier
of the NVM Express Base Specification are unlikely to retry commands aborted with the Command
Interrupted status code as that status code was introduced after NVM Express Base Specification, Revision
1.3). A host that supports that command retry behavior indicates its support to the controller by setting a
field to 1h in the Host Behavior Support Feature. Setting that field to 1h enables controller use of the
Command Interrupted status code, with the result that this status code is used only with hosts that have
indicated support for the associated command retry behavior.
This Feature is not saveable (refer to Figure 195). The default value of this Feature shall be all bytes cleared
to 0h.
After a successful completion of a Set Features command for this Feature, the controller may use controller-
to-host functionality that depends on specific host behavior as indicated by the attributes. If multiple Set
Features commands for this Feature are processed by the controller, only information from the most recent
successful command is retained (i.e., subsequent commands replace information provided by previous
commands).
If a Get Features command is submitted for this Feature, the attributes specified in Figure 407 are returned
in the data buffer for that command.
