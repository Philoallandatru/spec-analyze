---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 63
headings: []
---

# Source page 63

NVM Express® Base Specification, Revision 2.1

41
In the Connect command to a Discovery subsystem that provides a unique  Discovery Service NQN, the
host may use either of the following:
• the well-known Discovery Service NQN; or
• the unique Discovery Service NQN of that Discovery subsystem.
In the Connect command to a Discovery subsystem that does not provide a unique Discovery Service NQN,
the host uses the well-known Discovery Service NQN.
The method that a host uses to obtain the fabric information necessary to connect to a Discovery controller
using the well-known Discovery Service NQN or the unique NQN via the NVMe Transport may be:
a) implementation specific;
b) fabric specific;
c) known in advance (e.g., a well-known address);
d) administratively configured; or
e) for IP-based fabrics, Automated Discovery of Discovery Controllers for IP -based Fabrics (refer to
section 8.3.1) may be used.
The Discovery log page provided by a Discovery controller contains one or more entries. Each entry
specifies information necessary for the host to connect to an NVM subsystem. An entry may be associated
with an NVM subsystem that exposes namespaces or a referral to another Disco very Service. There are
no ordering requirements for log page entries within the Discovery log page.
Discovery controller(s) may provide different log page contents depending on the Host NQN provided (e.g.,
different NVM subsystems may be accessible to different hosts). The set of Discovery Log Page Entries
should include all applicable addresses on the same fabric as the Discovery Service and may include
addresses on other fabrics.
Discovery controllers that support explicit persistent connections shall support both the Asynchronous
Event Request command and the Keep Alive command (refer to sections 5.1.2 and 5.1.14 respectively). A
host requests an explicit persistent connection to a Discovery controller and Asynchronous Event
Notifications from the Discovery controller on that persistent connection by specifying a non -zero Keep
Alive Timer value in the Connect command. If the Connect command specifies a non-zero Keep Alive Timer
value and the Discovery controller does not support Asynchronous Events, then the Discovery controller
shall return a status value of Connect Invalid Parameters (refer to Figure 548) for the Connect command.
Discovery controllers shall indicate support for Discovery Log Change Notifications in the Identify Controller
data structure (refer to Figure 312).
Discovery controllers that do not support explicit persistent connections shall not support Keep Alive
commands and may use a fixed Discovery controller activity timeout value (e.g., 2 minutes). If no commands
are received by such a Discovery controller wi thin that time period, the controller may perform the actions
for Keep Alive Timer expiration defined in section 3.9.5.
A Discovery controller shall not support the Disconnect command.
A Discovery log page with multiple Discovery Log Page Entries for the same NVM subsystem indicates that
there are multiple fabric paths to the NVM subsystem, and/or that multiple static controllers may share a
fabric path. The host may use this information to form multiple associations to controllers within an  NVM
subsystem.
Multiple Discovery Log Page E ntries for the same NVM subsystem with different Port ID values indicates
that the resulting NVMe Transport connections are independent with respect to NVM subsystem port
hardware failures. A host that uses a single association should pick  a record to attach to an NVM
subsystem. A host that uses multiple associations should choose different ports.
A transport specific method may exist to indicate changes to a Discovery controller.
Controller IDs in the range FFF0h to FFFFh are not allocated as valid Controller IDs on completion of a
Connect command, as described in section 6.3. Figure 27 defines these Controller IDs.
