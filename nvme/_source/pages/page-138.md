---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 138
headings: []
---

# Source page 138

NVM Express® Base Specification, Revision 2.1

116
An abrupt NVM Subsystem Shutdown to this controller and all controllers within the associated domain is
initiated by:
• a host writing the value 41627074h ("Abpt") to NSSD.NSSC when CAP.CPS is set to 10b; or
• issuing an NVMe-MI Shutdown command to a Management Endpoint (refer to the NVM Express
Management Interface Specification) specifying an abrupt shutdown.
While NVM Subsystem Shutdown processing is in progress, any controller in the domain may abort any
command with a status code of Commands Aborted due to Power Loss Notification.
It is recommended that the host wait a minimum of the NVM Subsystem Shutdown Latency reported in the
Identify Controller data structure (refer to Figure 312) for NVM Subsystem Shutdown processing on a
domain to complete; if the reported NVM Subsystem Shutdown Latency value is 0h, then the host should
wait for a minimum of 30 seconds. While an NVM Subsystem Shutdown is reported as in progress, it is not
recommended to reset the domain via either an NVM Subsystem Reset on the domain or power cycling the
domain (which causes an NVM Subsystem Reset on the domain). This aborts the NVM Subsystem
Shutdown which may impact the subsequent time required for the domain t o become ready to perform I/O
(e.g., after power is reapplied following a power cycle).
For either a normal or an abrupt NVM Subsystem Shutdown on the domain, the domain is ready to be
powered off (e.g., the media is in the shutdown state (refer to Figure 85)) when the CSTS.ST bit is set to
‘1’ and the CSTS.SHST field indicates that shutdown processing is complete (i.e., the CSTS.SHST field is
set to 10b) on any controller in the domain. The NVM subsystem shall not set the CSTS.SHST field to 10b
on any controller in the domain until the entire domain is ready to be powered off. The NV M Subsystem
shall indicate that NVM Subsystem Shutdown processing is complete by setting the CSTS.SHST field to
10b on all controllers in the domain. The domain remains ready to be powered off (e.g., the media remains
in the shutdown state) until:
A. an NVM Subsystem Reset occurs on that domain; or
B. an Admin command that requires access to the media (refer to Figure 84) and specifies the Ignore
Shutdown bit set to ‘1’ is processed by any controller in the domain via the out-of-band mechanism
(refer to the NVM Express Management Interface Specification).
If a normal or an abrupt NVM Subsystem Shutdown is reported as in progress or is reported as complete
within a domain (i.e., the CSTS.ST bit is set to ‘1’ and the CSTS.SHST field is set to either 01b or 10b on
all controllers in the domain, indicating that  an NVM Subsystem Reset on the domain has not occurred
since initiation of that NVM Subsystem Shutdown), then:
• an NVM Subsystem Reset on the domain:
o shall abort any in progress NVM Subsystem Shutdown on the domain; and
o clears the CSTS.SHST field to 00b in all controllers in the domain; and
o clears the CSTS.ST bit to ‘0’ in all controllers in the domain;
and
• a Controller Level Reset of any controller in the domain that is initiated by any other method (refer
to section 3.7.2)
o shall not abort any in progress NVM Subsystem Shutdown on the domain;
o does not change the values of the CSTS.ST bit and the CSTS.SHST field, as described in
Figure 42; and
o shall not cause the domain to cease being ready to be powered off (e.g., shall not transition
the media out of the shutdown state) if the domain was ready to be powered off when that
Controller Level Reset was initiated.
To start executing commands on the controller after that controller reports NVM Subsystem Shutdown
processing complete (i.e., the CSTS.ST bit is set to ‘1’ and the CSTS.SHST field is set to 10b):
• regardless of the value of CC.EN, an NVM Subsystem Reset on that domain is required; and
• the CC.EN bit is subsequently required to be set to ‘1’ as part of the initialization sequence (refer
to section 3.5).
