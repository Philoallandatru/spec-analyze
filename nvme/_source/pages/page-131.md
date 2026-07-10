---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 131
headings: ["3.5.4.1 Handling Errors During Initialization"]
---

# Source page 131

NVM Express® Base Specification, Revision 2.1

109
i. the Controller Ready Independent of Media Timeout (CRTO.CRIMT) field is not applicable;
and
ii. the Controller Ready With Media Timeout (CRTO.CRWMT) indicates the worst -case time
the host should wait after the controller is enabled for:
1. the controller to become ready and be able to process all commands without error
as described in section 3.5.4.1; and
2. all attached namespaces and media required to process Admin commands to
become ready,
and
d) If the controller is in Controller Ready Independent of Media mode (i.e., the CC.CRIME bit is set to
‘1’), then
i. the Controller Ready With Media Timeout (CRTO.CRWMT) field indicates the worst-case
time that host software should wait for all attached namespaces and media required to
process Admin commands to become ready after the controller is enabled; and
ii. the Controller Ready Independent of Media Timeout (CRTO.CRIMT) field indicates the
worst-case time the host should wait after the controller is enabled for the controller to
become ready and be able to process:
1. all commands that do not access attached namespaces; and
2. Admin commands that do not require access to media,
without error as described in section 3.5.4.1.
Changes to the value of the CC.CRIME bit shall have no effect on the values of the CRTO.CRWMT and
CRTO.CRIMT fields. Changes to the value of the CC.CRIME bit may have an effect on the value of the
CAP.TO field (refer to Figure 36).
 Handling Errors During Initialization
If the CAP.CRMS field is non-zero and the controller has been enabled by transitioning CC.EN from ‘0’ to
‘1’ and the controller encounters a failure that prevents:
a) at least one:
• command that does not access attached namespaces; or
• Admin command that does not require access to media (refer to Figure 84),
from being able to be processed without error by the amount of time indicated by the:
• Controller Ready Independent of Media Timeout (CRTO.CRIMT) field since the controller
was enabled if the controller is in Controller Ready Independent of Media mode (i.e., the
CC.CRIME bit is set to ‘1’); or
• Controller Ready With Media Timeout (CRTO.CRWMT) field since the controller was
enabled if the controller is in Controller Ready With Media mode (i.e., the CC.CRIME bit is
cleared to ‘0’);
b) at least one namespace attached to the controller from becoming ready by the amount of time
indicated by the Controller Ready With Media Timeout ( CRTO.CRWMT) field since the controller
was enabled; or
c) media required by at least one Admin command from becoming ready by the amount of time
indicated by the Controller Ready With Media Timeout ( CRTO.CRWMT) field since the controller
was enabled,
then:
a) if the controller has not become ready, then the controller shall become ready (i.e., set CSTS.RDY
to ‘1’) no later than CRTO.CRWMT amount of time after the controller was enabled; and
