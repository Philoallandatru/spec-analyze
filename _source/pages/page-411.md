---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 411
headings: ["5.1.25.1.18 Spinup Control (Feature Identifier 1Ah)", "5.1.25.1.19 Power Loss Signaling Config (Feature Identifier 1Bh)"]
---

# Source page 411

NVM Express® Base Specification, Revision 2.1

389
Figure 410: I/O Command Set Profile – Command Dword 11
Bits Description
08:00
I/O Command Set Combination Index (IOCSCI): This field specifies the index of the I/O Command Set
Combination that is to be used. This field is used for the Set Features command only and is ignored for
the Get Features command for this Feature.
The controller shall abort a command that specifies an index that corresponds to an I/O Command Set
Combination that has a value of 0h with a status code of I/O Command Set Combination Rejected.
If a Get Features command is submitted for this Feature, then the attributes described in Figure 411 are
returned in Dword 0 of the completion queue entry for that command.
Figure 411: I/O Command Set Profile – Completion Queue Entry Dword 0
Bits Description
31:09 Reserved
08:00 I/O Command Set Combination Index (IOCSCI): This field returns the index of the currently selected
I/O Command Set Combination.
5.1.25.1.18 Spinup Control (Feature Identifier 1Ah)
This Feature allows the host to configure the method for initial spinup for Endurance Groups that store data
on rotational media (refer to section 8.1.23).
If the NVM subsystem does not contain any Endurance Groups that store data on rotational media, then
the controller shall abort the Set Features command and the Get Features command for this Feature with
status code of Invalid Field in Command.
The method is specified in Command Dword 11 (refer to Figure 412).
Figure 412: Spinup Control – Command Dword 11
Bits Description
31:01 Reserved
0 Spinup Control Enable (SCE): If this bit is set to ‘1’, then the Spinup Control feature is enabled. If this
bit is cleared to ‘0’, then the Spinup Control feature is disabled. The setting is persistent.
If a Get Features command is submitted for this Feature, the attributes described in (refer to Figure 413)
are returned in Dword 0 of the completion queue entry for that command.
Figure 413: Completion Queue Entry Dword 0
Bits Description
31:01 Reserved
0 Spinup Control State (SCS): If this bit is set to ‘1’, then the Spinup Control feature is enabled. If this bit
is cleared to ‘0’, then the Spinup Control feature is disabled.
5.1.25.1.19 Power Loss Signaling Config (Feature Identifier 1Bh)
This Feature configures the behavior of Power Loss Signaling (refer to section 8.2.5). The scope of this
Feature is as described in Figure 385. The attributes are specified in Command Dword 11 (refer to Figure
414).
If a Get Features command is successfully completed for this Feature, then the attribute described in Figure
414 is returned in Dword 0 of the completion queue entry for that command.
If a Set Features command is submitted for this Feature and the Power Loss Signaling Mode field specifies
a Power Loss Signaling mode that is not supported (refer to the Power Loss Signaling Information field in
Figure 312), then that command shall be aborted with a status code of Invalid Field in Command.
