---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 620
headings: ["8.2.5.1.6 EPF Complete Port Disabled State", "8.2.5.1.7 EPF Processing Port Enabled State", "8.2.5.1.8 EPF Complete Port Enabled State", "8.2.5.2 Forced Quiescence Processing"]
---

# Source page 620

NVM Express® Base Specification, Revision 2.1

598
Figure 666: EPF Processing Port Disabled State Transition Conditions
State Transitions Transition Condition Starting Ending
EPF Processing
Port Disabled
EPF Complete
Port Disabled • The controller completes EPF processing.
8.2.5.1.6 EPF Complete Port Disabled State
In the EPF Complete Port Disabled state, the controller has completed Emergency Power Fail Processing.
The port is disabled. The following are not able to be initiated through the port:
a) Controller Level Reset;
b) controller shutdown;
c) NVM Subsystem Reset; and
d) NVM Subsystem Shutdown.
Transitions out of this state are defined in Figure 667.
Figure 667: EPF Complete Port Disabled State Transition Conditions
State Transitions Transition Condition Starting Ending
EPF Complete Port
Disabled PLS Not Ready • Power is cycled.
8.2.5.1.7 EPF Processing Port Enabled State
In the EPF Processing Port Enabled state, the controller is performing Emergency Power Fail Processing,
as described in section 8.2.5.3. The port is enabled.
Transitions out of this state are defined in Figure 668.
Figure 668: EPF Processing Port Enabled State Transition Conditions
State Transitions Transition Condition Starting Ending
EPF Processing
Port Enabled
EPF Complete
Port Enabled • The controller completes EPF processing.
PLS Not Ready • The controller processes a Controller Level Reset; or
• CSTS.SHST is not cleared to 00b.
8.2.5.1.8 EPF Complete Port Enabled State
In the EPF Complete Port Enabled state, the controller has completed Emergency Power Fail Processing.
The port is enabled.
Transitions out of this state are defined in Figure 669.
Figure 669: EPF Complete Port Enabled State Transition Conditions
State Transitions Transition Condition Starting Ending
EPF Complete Port
Enabled PLS Not Ready • The controller processes a Controller Level Reset; or
• CSTS.SHST is not cleared to 00b.
 Forced Quiescence Processing
This section describes the behavior of each controller in the domain when the domain is configured for
Power Loss Signaling with Forced Quiescence (refer to section 5.1.25.1.19).
