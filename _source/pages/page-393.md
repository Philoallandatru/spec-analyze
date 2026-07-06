---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 393
headings: []
---

# Source page 393

NVM Express® Base Specification, Revision 2.1

371
Figure 383: Set Features – Command Dword 10
Bits Description
31
Save (SV):  This bit specifies that the controller shall save the attribute so that the attribute persists
through all power states and resets.
The controller indicates in the Save and Select Feature Support (SSFS) bit in the Optional NVM
Command Support fi eld of the Identify Controller d ata structure in Figure 312 whether this bit is
supported.
If the Feature Identifier specified in the Set Features command is not saveable by the controller and the
controller receives a Set Features command with this bit set to ‘1’, then the command shall be aborted
with a status code of Feature Identifier Not Saveable.
30:08 Reserved
07:00 Feature Identifier (FID): This field indicates the identifier of the Feature that attributes are being
specified for.
If the controller supports selection of a UUID by the Set Features command (refer to Figure 385 and section
8.1.28) and the controller supports selection of a UUID for the specified vendor specific feature identifier
(refer to Figure 385), then Command Dword 14 is used to specify a UUID Index value (refer to Figure 384).
If the controller does not support selection of a UUID by the Set Features command or the controller does
not support selection of a UUID for the specified vendor specific feature identifier, then Command Dword
14 does not specify a UUID Index value.
Figure 384: Set Features – Command Dword 14
Bits Description
31:07 Reserved
06:00 UUID Index (UIDX): Refer to Figure 658.
Figure 385 defines the Features that are able to be configured with a Set Features command and retrieved
with a Get Features command.
Section 5.1.25.1 describes features that are common to all transport models . Section 5.1.25.2 describes
features that are specific to the Memory -based transport model . Section 5.1.25.3 describes features that
are specific to the Message-based transport model.
Figure 385: Set Features – Feature Identifiers
Feature
Identifier
Current Setting
Persists
Across Power
Cycle and
Reset 2
Uses
Memory
Buffer for
Attributes
Feature Name Scope 6
00h Reserved
01h No No Arbitration Controller
02h No No Power Management Controller 7
03h Refer to the NVM Command Set Specification
04h No No Temperature Threshold Controller
05h Refer to the NVM Command Set Specification
06h No No Volatile Write Cache Controller
07h No No Number of Queues Controller
08h No No Interrupt Coalescing Controller
09h No No Interrupt Vector Configuration Controller
0Ah Refer to the NVM Command Set Specification
0Bh No No Asynchronous Event Configuration Controller
0Ch No Yes Autonomous Power State Transition Controller 7
