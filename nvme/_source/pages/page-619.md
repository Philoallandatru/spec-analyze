---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 619
headings: ["8.2.5.1.3 FQ Processing State", "8.2.5.1.4 FQ Complete State", "8.2.5.1.5 EPF Processing Port Disabled State"]
---

# Source page 619

NVM Express® Base Specification, Revision 2.1

597
Figure 663: PLS Ready State Transition Conditions
State Transitions Transition Condition Starting Ending
PLS Ready
FQ Processing • The controller is configured for FQ processing; and
• the PLN variable transitions from Deasserted to Asserted.
EPF Processing
Port Disabled
• The controller is configured for EPF processing;
• the controller implements the EPF Processing Port Disabled state;
and
• the PLN variable transitions from Deasserted to Asserted.
EPF Processing
Port Enabled
• The controller is configured for EPF processing;
• the controller implements the EPF Processing Port Enabled state;
and
• the PLN variable transitions from Deasserted to Asserted.
PLS Not Ready • The controller processes a Controller Level Reset; or
• CSTS.SHST is not cleared to 00b.
8.2.5.1.3 FQ Processing State
In the FQ Processing state, the controller is performing Forced Quiescence Processing, as described in
section 8.2.5.2.
Transitions out of this state are defined in Figure 664.
Figure 664: FQ Processing State Transition Conditions
State Transitions Transition Condition Starting Ending
FQ Processing
FQ Complete • The controller completes FQ processing.
PLS Not Ready • The controller processes a Controller Level Reset; or
• CSTS.SHST is not cleared to 00b.
8.2.5.1.4 FQ Complete State
In the FQ Complete state, the controller has completed Forced Quiescence Processing.
Transitions out of this state are defined in Figure 665.
Figure 665: FQ Complete State Transition Conditions
State Transitions Transition Condition Starting Ending
FQ Complete PLS Not Ready • The controller processes a Controller Level Reset; or
• CSTS.SHST is not cleared to 00b.
PLS Ready • The PLN variable is set to Deasserted.
8.2.5.1.5 EPF Processing Port Disabled State
In the EPF Processing Port Disabled state, the controller is performing Emergency Power Fail Processing,
as described in section 8.2.5.3. The port is disabled. The following are not able to be initiated through the
port:
a) Controller Level Reset;
b) controller shutdown;
c) NVM Subsystem Reset; and
d) NVM Subsystem Shutdown.
Transitions out of this state are defined in Figure 666.
