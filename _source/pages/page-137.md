---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 137
headings: ["3.6.3.2 Domain Shutdown in a Multiple Domain NVM Subsystem"]
---

# Source page 137

NVM Express® Base Specification, Revision 2.1

115
For either a normal or an abrupt NVM Subsystem Shutdown, the NVM subsystem is ready to be powered
off (e.g., the media is in the shutdown state (refer to Figure 85)) when the CSTS.ST bit is set to ‘1’ and the
CSTS.SHST field indicates that shutdown processing is complete (i.e., the CSTS.SHST field is set to 10b)
on any controller in the NVM subsystem. The NVM subsystem shall not set the CSTS.SHST field to 10b on
any controller in the NVM subsystem until the entire NVM subsystem is ready to be powered off. The NVM
Subsystem shall indicate that NVM Subsystem Shutdown processing is complete by setting the
CSTS.SHST field to 10b on all controllers in the NVM subsystem. T he NVM subsystem remains ready to
be powered off (e.g., the media remains in the shutdown state) until:
A. an NVM Subsystem Reset; or
B. an Admin command that requires access to the media (refer to Figure 84) and specifies the Ignore
Shutdown bit set to ‘1’ is processed by any controller via the out -of-band mechanism (refer to the
NVM Express Management Interface Specification).
If a normal or an abrupt NVM Subsystem Shutdown is reported as in progress or is reported as complete
within the NVM subsystem (i.e., the CSTS.ST bit is set to ‘1’ and the CSTS.SHST field is set to either 01b
or 10b on all controllers in the NVM subsystem, indicating that an NVM Subsystem Reset has not occurred
since initiation of that NVM Subsystem Shutdown), then:
• an NVM Subsystem Reset:
o shall abort any in progress NVM Subsystem Shutdown;
o clears the CSTS.SHST field to 00b in all controllers in the NVM subsystem; and
o clears the CSTS.ST bit to ‘0’ in all controllers in the NVM subsystem;
and
• a Controller Level Reset of any controller in the NVM subsystem that is initiated by any other
method (refer to section 3.7.2):
o shall not abort any in progress NVM Subsystem Shutdown;
o does not change the values of the CSTS.ST bit and the CSTS.SHST field, as described in
Figure 40; and
o shall not cause that NVM subsystem to cease being ready to be powered off (e.g., shall
not transition the media out of the shutdown state) if that NVM Subsystem was ready to be
powered off when that Controller Level Reset was initiated.
To start executing commands on the controller after that controller reports NVM Subsystem Shutdown
processing complete (i.e., the CSTS.ST bit is set to ‘1’ and the CSTS.SHST field is set to 10b):
• regardless of the value of the CC.EN bit, an NVM Subsystem Reset is required; and
• the CC.EN bit is subsequently required to be set to ‘1’ as part of the initialization sequence (refer
to section 3.5).
The initialization sequence (refer to section 3.5) should then be executed on that controller.
 Domain Shutdown in a Multiple Domain NVM Subsystem
A normal NVM Subsystem Shutdown on this controller and all controllers within the associated domain is
initiated by:
• a host writing the value 4E726D6Ch ("Nrml") to NSSD.NSSC when CAP.CPS is set to 10b; or
• issuing an NVMe-MI Shutdown command to a Management Endpoint (refer to the NVM Express
Management Interface Specification) specifying a normal shutdown.
For each controller in the domain for this normal NVM subsystem shutdown, if:
• CSTS.SHST is cleared to 00b; and
• An outstanding Asynchronous Event Request command exists,
then the controller shall issue a Normal NVM Subsystem Shutdown event prior to shutting down the
controller.
