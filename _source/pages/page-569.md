---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 569
headings: ["8.1.21 Replay Protected Memory Block"]
---

# Source page 569

NVM Express® Base Specification, Revision 2.1

547
Figure 632: Read Recovery Level Overview
Level O/M Definition
0 O  Maximum
Recovery
1 O
Decreasing Amount of Recovery

2 O
3 O
4 M Default
5 O
6 O
7 O
8 O
9 O
10 O
11 O
12 O
13 O
14 O
15 M Fast Fail Minimum
Recovery
If Read Recovery Levels are supported, then the NVM subsystem and all controllers shall:
• Support at least Level 4 and Level 15;
• Indicate support for Read Recovery Levels in the Controller Attributes field in the Identify Controller
data structure;
• Support the Read Recovery Levels Supported field in the Identify Controller data structure; and
• Support the Read Recovery Level Config Feature.
 Replay Protected Memory Block
The Replay Protected Memory Block (RPMB) provides a means for the system to store data to a specific
memory area in an authenticated and replay protected manner. This is provided by first programming
authentication key information to the controller that is  used as a shared secret. The system is not
authenticated in this phase, therefore the authentication key programming should be done in a secure
environment (e.g., as part of the manufacturing process). The authentication key is utilized to sign the read
and write accesses made to the replay protected memory area with a Message Authentication Code (MAC).
Use of random number (nonce) generation and a write count property provide additional protection against
replay of messages where messages could be recorded and played back later by an attacker.
Any attempt to access the replay protected memory area prior to the Authentication Key being programmed
results in an RPMB Operation Result Operation Status field set to 07h (i.e., Authentication Key not yet
