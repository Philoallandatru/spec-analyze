---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 130
headings: ["3.5.4 Controller Ready Timeouts During Initialization"]
---

# Source page 130

NVM Express® Base Specification, Revision 2.1

108
Figure 84: Admin Commands Permitted to Return a Status Code of Admin Command Media Not
Ready
Admin Command Additional Restrictions
Security Receive1
Security Send1
Vendor Specific
Notes:
1. A host may require discovery operations performed via Security Send/Receive (e.g., TCG Level 0 Discovery) to
be processed prior to media being ready . Therefore, it is recomm ended that controllers not return Admin
Command Media Not Ready for such discovery operations.
The Controller Ready Modes Supported (CAP.CRMS) field (refer to Figure 36) indicates which controller
ready modes are supported. The CAP.CRMS field consists of two bits:
• the Controller Ready With Media Support (CAP.CRMS.CRWMS) bit; and
• the Controller Ready Independent of Media Support (CAP.CRMS.CRIMS) bit.
Controllers shall set the CAP.CRMS.CRWMS bit to ‘1’ (i.e., set the CAP.CRMS field to 01b or 11b). The
CAP.CRMS.CRWMS bit was not defined prior to NVM Express Base Specification , R evision 2.0.
Controllers compliant with revisions earlier than NVM Express Base Specification, Revision 2.0 may clear
the CAP.CRMS.CRWMS field to 00b.
The Controller Ready Independent of Media Enable (CC.CRIME) bit (refer to Figure 41) controls the
controller ready mode based on the value of the CAP.CRMS field as follows:
a) If the CAP.CRMS field is cleared to 00b, the controller ready mode is not able to be selected. In
this case, the read -only CC.CRIME bit shall be cleared to ‘0’ and should be ignored by host
software;
b) If the CAP.CRMS field is set to 01b (i.e., the CAP.CRMS.CRIMS bit is cleared to ‘0’ and the
CAP.CRMS.CRWMS bit is set to ‘1’), then the controller is in Controller Ready With Media mode
and the read-only CC.CRIME bit shall be cleared to ‘0’; and
c) If the CAP.CRMS field is set to 11b, then both controller ready modes are supported, and the host
may select the controller ready mode by modifying the value of the CC.CRIME bit. In this situation,
the host should set the controller ready mode by writing to the CC.CRIME bit before the controller
is enabled (e.g., as part of the initialization sequence of actions described in section 3.5).
 Controller Ready Timeouts During Initialization
The CAP.CRMS field was not defined prior to NVM Express Base Specification , Revision 2.0. Controllers
compliant with  revisions earlier than  NVM Express Base Specification , Revision 2.0 may clear the
CAP.CRMS field to 00b. This section is applicable to controllers that clear the CAP.CRMS field to 00b and
controllers that set CAP.CRMS to a non-zero value.
There are three controller ready timeout fields:
1. CAP.TO (refer to Figure 36);
2. CRTO.CRWMT (refer to Figure 57); and
3. CRTO.CRIMT (refer to Figure 57).
The details regarding these timeouts during controller initialization are as follows:
a) The CAP.TO field shall be set as described in Figure 36;
b) If the CAP.CRMS field is cleared to 00b’, then the worst -case time the host should wait after the
controller is enabled (i.e., CC.EN transitions from ‘0’ to ‘1’) for the controller to become ready
(CSTS.RDY transitions from ‘0’ to ‘1’) is indicated by CAP.TO;
c) If the controller is in Controller Ready With Media mode (i.e., the CC.CRIME bit is cleared to ‘0’),
then:
