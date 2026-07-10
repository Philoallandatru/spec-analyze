---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 136
headings: ["3.6.3.1 NVM Subsystem Shutdown in a Single Domain NVM Subsystem"]
---

# Source page 136

NVM Express® Base Specification, Revision 2.1

114
If a controller clears the CAP.NSSES bit to ‘0’, then, as defined in revision 2.0 of the NVM Express Base
Specification:
a. while an NVM Subsystem Shutdown is reported as in progress or is reported as complete, a
Controller Reset does not initiate a CLR (i.e., Controller Reset is disabled); and
b. while an NVM Subsystem Shutdown is reported as complete, any CLR initiated by any transport -
specific reset type may clear the value of the CSTS.ST bit to ‘0’ and may clear the value of the
CSTS.SHST field to 00b.
A host is able to support NVM Subsystem Shutdown functionality both on controllers that set the
CAP.NSSES bit to ‘1’ and on controllers that clear the CAP.NSSES bit to ‘0’ by ensuring that any NVM
Subsystem Shutdown is followed by an NVM Subsystem Reset regardless of the value of the CSTS.ST bit
and the value of the CSTS.SHST field.
 NVM Subsystem Shutdown in a Single Domain NVM Subsystem
A normal shutdown on all controllers within the NVM subsystem (i.e., normal NVM Subsystem Shutdown)
is initiated by:
• a host writing the value 4E726D6Ch ("Nrml") to NSSD.NSSC when CAP.CPS is set to 11b; or
• issuing an NVMe-MI Shutdown command to a Management Endpoint (refer to the NVM Express
Management Interface Specification) specifying a normal shutdown.
For each controller in the NVM subsystem for this normal NVM Subsystem Shutdown, if:
• CSTS.SHST is set to 00b; and
• An outstanding Asynchronous Event Request command exists,
then the controller shall issue a Normal NVM Subsystem Shutdown event prior to shutting down the
controller.
An abrupt shutdown on all controllers within the NVM subsystem (i.e., abrupt NVM Subsystem Shutdown)
is initiated by:
• a host writing the value 41627074h ("Abpt") to NSSD.NSSC when CAP.CPS is set to 11b; or
• issuing an NVMe-MI Shutdown command to a Management Endpoint (refer to the NVM Express
Management Interface Specification) specifying an abrupt shutdown.
While NVM Subsystem Shutdown processing is in progress, any controller in the NVM subsystem may
abort any command with a status code of Commands Aborted due to Power Loss Notification.
It is recommended that the host wait a minimum of the NVM Subsystem Shutdown Latency reported in the
Identify Controller data structure (refer to Figure 312) for NVM Subsystem Shutdown processing to
complete; if the reported NVM Subsystem Shutdown Latency value is 0h, then the host should wait for a
minimum of 30 seconds. While an NVM Subsystem Shutdown is reported as in progress, it is not
recommended to reset the NVM subsystem via an NVM Subsystem Reset or a power cycle (which causes
an NVM Subsystem Reset). This aborts the NVM Subsystem Shutdown which may impact the subsequent
time required for the NVM subsystem to become ready to perform I/O (e.g., after p ower is reapplied
following a power cycle).
