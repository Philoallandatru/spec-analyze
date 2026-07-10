---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 132
headings: ["3.6 Shutdown Processing"]
---

# Source page 132

NVM Express® Base Specification, Revision 2.1

110
b) if the Persistent Event log page is supported, then the controller shall record a n NVM Subsystem
Hardware Error Event with the NVM Subsystem Hardware Error Event code set to a value of
Controller Ready Timeout Exceeded in the Persistent Event log page (refer to Figure 236).
3.6 Shutdown Processing
This section describes the recommended procedure for shutdown processing prior to a power-off condition.
There are two shutdown processing mechanisms, controller shutdown (refer to sections 3.6.1 and 3.6.2)
and NVM Subsystem Shutdown (refer to section 3.6.3). The CSTS.ST bit indicates the shutdown
mechanism that is in progress, if any (refer to Figure 42). A host requests a controller shutdown by modifying
the CC.SHN field (refer to  Figure 41). A host requests an NVM Subsystem Shutdown by modifying the
NSSD property (refer to section 3.1.4.20) or by issuing an NVMe-MI Shutdown command to a Management
Endpoint (refer to the NVM Express Management Interface Specification).
At most one shutdown processing mechanism is able to be in progress for a controller at any time. If an
NVM Subsystem Shutdown is requested while a controller shutdown is in progress, then the NVM
Subsystem Shutdown overrides the controller shutdown. The p rogress and completion of shutdown
processing is indicated by the CSTS.SHST field (refer to Figure 42).
NVM Subsystem Shutdown should not be supported by any NVM subsystem that does not support more
than one controller, without counting virtual controllers (e.g., NVM Subsystem Shutdown should not be
supported by an NVM subsystem that supports one primary con troller and multiple secondary controllers)
(refer to section 8.2.6).
Figure 85 describes the interactions of the shutdown processing state indicated by the CSTS.SHST field
with the state of the controller indicated by the CC.EN bit (refer to Figure 41) and by the CSTS.RDY bit
(refer to Figure 42). The four possible media states in Figure 85 are: shutdown, shutdown in progress,
usable, and initialization in progress.
Figure 85: Shutdown Processing Interactions
CC.EN CSTS.RDY

CSTS.SHST
Controller able to
process Admin and I/O
commands4
Media state Controller ready to
be powered off
0 0 00b no any
implementation
specific1
0 0 01b no3 shutdown in
progress no
0 0 10b no shutdown yes
1 1 00b yes
initialization in
progress or
usable2
no
1 1 01b no3 shutdown in
progress no
1 1 10b no shutdown yes
Notes:
1. In some cases (e.g., following initial application of power, or following a Controller Level Reset that occurred
while shutdown processing was reported as complete), the controller is permitted to initialize the media and
cease being ready to be powered off as a consequence.
2. If the CC.CRIME bit is cleared to ‘0’, then the media is usable. If the CC.CRIME bit is set to ‘1’, then either
media initialization is in progress or the media is usable (refer to Figure 41).
3. While shutdown processing is in progress, the controller may abort any command with a status code of
Commands Aborted due to Power Loss Notification.
4. I/O commands are only able to be processed by a controller that supports I/O commands. Fabrics commands
are always able to be processed by a controller that supports Fabrics commands.
