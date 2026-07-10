---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 270
headings: ["5.1.12.1.14.2.11 Set Feature Event (Event Type 0Bh)"]
---

# Source page 270

NVM Express® Base Specification, Revision 2.1

248
5.1.12.1.14.2.11 Set Feature Event (Event Type 0Bh)
The Set Feature Event persists the data of a successful Set Features command. The event contains a
Persistent Event Log Event header and the Set Feature Event data (refer to Figure 246).
The Set Feature Event shall set the Persistent Event Log Event Format Header:
• Event Type field to 0Bh; and
• Event Type Revision Field to 01h.
A Set Feature Event shall be recorded in the Persistent Event Log when the following criteria are met:
a) A Set Features command completes successfully;
b) The Feature Identifier in that Set Features command  is supported to be logged in the Persistent
Event Log; and
c) There is a change to the controller settings for the Feature Identifier in that Set Features command
(i.e., the same setting is not set again).
A Set Feature Event may be recorded in the Persistent Event Log when there is no change to the controller
settings for the Feature Identifier in that Set Features command if the following criteria are met:
a) A Set Features command completes successfully; and
b) The Feature Identifier in that Set Features command  is supported to be logged in the Persistent
Event Log.
The Feature Identifiers that may be supported to be logged in the Persistent Event Log are shown in Figure
245.
Figure 245: Feature Persistent Event Logging Requirements
Feature Name Feature
Identifier
Controller PE Logging Requirements 1
I/O Admin Discovery
Arbitration 01h O P P
Power Management 02h NR NR P
 03h Refer to the applicable I/O Command Set specification
Temperature Threshold 04h O O P
 05h Refer to the applicable I/O Command Set specification
Volatile Write Cache 06h O P P
Number of Queues 07h O P P
Interrupt Coalescing 08h O O P
Interrupt Vector Configuration 09h O O P
 0Ah Refer to the applicable I/O Command Set specification
Asynchronous Event Configuration 0Bh NR NR NR
Autonomous Power State Transition 0Ch O O P
Host Memory Buffer 0Dh O O P
Timestamp 0Eh P P P
Keep Alive Timer 0Fh O O O
Host Controlled Thermal Management 10h O O P
Non-Operational Power State Config 11h O O P
Read Recovery Level Config 12h O O P
Predictable Latency Mode Config 13h O P P
Predictable Latency Mode Window 14h P O P
 15h Refer to the applicable I/O Command Set specification
Host Behavior Support 16h O O P
Sanitize Config 17h O O P
Endurance Group Event Configuration 18h O O P
I/O Command Set Profile 19h O P P
Spinup Control 1Ah O P P
Power Loss Signaling Config 1Bh ? ? ?
 1Ch Refer to the applicable I/O Command Set specification
