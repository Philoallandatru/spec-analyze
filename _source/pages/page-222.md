---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 222
headings: ["5.1.11.1 Select field"]
---

# Source page 222

NVM Express® Base Specification, Revision 2.1

200
Figure 194: Get Features – Feature Identifiers
Description Feature
Identifier
Section Defining
Format of Attributes
Returned
Interrupt Coalescing 08h 5.1.25.2.2
Interrupt Vector Configuration 09h 5.1.25.2.3
Asynchronous Event Configuration 0Bh 5.1.25.1.5
Autonomous Power State Transition 0Ch 5.1.25.1.6
Host Memory Buffer 0Dh 5.1.25.2.4
Timestamp 0Eh 5.1.25.1.7
Keep Alive Timer 0Fh 5.1.25.1.8
Host Controlled Thermal Management 10h 5.1.25.1.9
Non-Operational Power State Config 11h 5.1.25.1.10
Read Recovery Level Config 12h 5.1.25.1.11
Predictable Latency Mode Config 13h 5.1.25.1.12
Predictable Latency Mode Window 14h 5.1.25.1.13
Host Behavior Support 16h 5.1.25.1.14
Sanitize Config 17h 5.1.25.1.15
Endurance Group Event Configuration 18h 5.1.25.1.16
I/O Command Set Profile 19h 5.1.25.1.17
Spinup Control 1Ah 5.1.25.1.18
Power Loss Signaling Config 1Bh 5.1.25.1.19
Flexible Data Placement  1Dh 5.1.25.1.20
Flexible Data Placement Events 1Eh 5.1.25.1.21
Namespace Admin Label 1Fh 5.1.25.1.22
Controller Data Queue 21h 5.1.25.1.23
Embedded Management Controller Address 78h 5.1.25.1.24
Host Management Agent Address 79h 5.1.25.1.25
Enhanced Controller Metadata1 7Dh 5.1.25.1.26.1
Controller Metadata1 7Eh 5.1.25.1.26.2
Namespace Metadata1 7Fh 5.1.25.1.26.3
Software Progress Marker 80h 5.1.25.1.27
Host Identifier 81h 5.1.25.1.28
Reservation Notification Mask 82h 5.1.25.1.29
Reservation Persistence 83h 5.1.25.1.30
Namespace Write Protection Config 84h 5.1.25.1.31
Boot Partition Write Protection Config 85h 5.1.25.1.32
I/O Command Set specific features  I/O Command Set
specification
Notes:
1. Information that is common to all Host Metadata features is described in section
5.1.25.1.26.
 Select field
A Select field cleared to 000b (i.e., current) returns the current operating attribute value for the Feature
Identifier specified.
A Select field set to 001b (i.e., default) returns the default attribute value for the Feature Identifier specified.
A Select field set to 010b (i.e., saved) returns the last saved attribute value for the Feature Identifier
specified (i.e., the last Set Features command completed without error, with the Save bit set to ‘1’ for the
Feature Identifier specified).
