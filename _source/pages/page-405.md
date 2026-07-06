---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 405
headings: ["5.1.25.1.11 Read Recovery Level Config (Feature Identifier 12h)", "5.1.25.1.12 Predictable Latency Mode Config (Feature Identifier 13h)"]
---

# Source page 405

NVM Express® Base Specification, Revision 2.1

383
Figure 399: Non-Operational Power State Config – Command Dword 11
Bits Description
00
Non-Operational Power State Permissive Mode Enable (NOPPME): If this bit is set to ‘1’ , then the
controller may temporarily exceed the power limits of any non-operational power state, up to the limits of
the last operational power state, to run controller-initiated background operations in that state (i.e., Non-
Operational Power State Permissive Mode is enabled). If this bit is cleared to ‘0’, then the controller shall
not exceed the limits of any non -operational state while running controller-initiated background
operations in that state (i.e., Non-Operational Power State Permissive Mode is disabled).
If Non-Operational Power State Permissive Mode is disabled, then:
a) thermal management that requires power (e.g., cooling fans) may be disabled; and
b) performance after resuming from the non -operational power state may be degraded until
background activity that was not allowed while in that non -operational power state has
completed.
If the host attempts to set this bit to ‘1’ and the controller does not support Non-Operational Power State
Permissive Mode as indicated in the Controller Attributes  (CTRATT) field of the Identify Controller data
structure, then the controller shall abort the command with a status code of Invalid Field in Command.
The Non-Operational Power State Config feature may interact with the Autonomous Power State Transition
feature (refer to section 5.1.25.1.6). Figure 394 shows these interactions.
5.1.25.1.11 Read Recovery Level Config (Feature Identifier 12h)
This Feature is used to configure the Read Recovery Level (refer to section 8.1.20). The attributes are
specified in Command Dword 11 and Command Dword 12. Modifying the Read Recovery Level has no
effect on the data contained in any associated namespace.
The scope of this Feature is:
• NVM Set if NVM Sets are supported; or
• NVM Subsystem if NVM Sets are not supported.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 401 are returned
in Dword 0 of the completion queue entry for that command.
Figure 400: Read Recovery Level Config – Command Dword 11
Bits Description
31:16 Reserved
15:00 NVM Set Identifier (NVMSETID): This field specifies the NVM Set to be modified. If NVM Sets are not
supported, then this field is ignored and the command applies to all namespaces in the NVM subsystem.

Figure 401: Read Recovery Level Config – Command Dword 12
Bits Description
31:04 Reserved
03:00 Read Recovery Level (RRL): This field sets the Read Recovery Level for the NVM Set specified.
5.1.25.1.12 Predictable Latency Mode Config (Feature Identifier 13h)
This Feature configures an NVM Set to use Predictable Latency Mode, including warning event thresholds.
Predictable Latency Mode and events are disabled by default. The attributes are specified in Command
Dword 11, Command Dword 12, and the Deterministic Threshold Configuration data structure.
The NVM Set has transitioned to Predictable Latency Mode when the controller completes a Set Features
command successfully with the Predictable Latency Enable bit in Command Dword 12 set to ‘1’. A transition
to the Predictable Latency Mode may be delayed ( i.e., the Set Features command completion is delayed)
if the NVM subsystem needs to perform background operations on the NVM in order to operate in
