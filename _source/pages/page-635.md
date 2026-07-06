---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 635
headings: []
---

# Source page 635

NVM Express® Base Specification, Revision 2.1

613
Upon completion of acknowledging a pull registration request (e.g., after a KDResp NVMe/TCP PDU (refer
to the Kickstart Discovery Response PDU section in the NVMe TCP Transport Specification) has been sent
by the Centralized Discovery controller (CDC)), the CDC performs the pull registration by:
1. sending a Connect command to the Direct Discovery controller (DDC) to establish an NVMe
connection with that DDC; and
2. sending a Get Log Page command to the DDC with the Log Page Identifier (LID) field set to 70h to
retrieve NVM subsystem discovery information contained in the associated Discovery log page
from that DDC. The CDC may:
a. set the Port Local Entries Only (PLEO) bit to ‘1’ in the Log Specific Parameter (LSP) field in
that Get Log Page command to request that NVM subsystem discovery information entries for
only NVM subsystem ports that are presented through the same NVM subsys tem port on the
DDC that received the Get Log Page command be returned; and
b. set the All NVM Subsystem Entries (ALLSUBE) bit to ‘1’ in the LSP field in that Get Log Page
command to request that records for all NVM subsystem ports be returned.
The DDC should match the NQN contained in the CDC NVMe Qualified Name (CDCNQN) field of the
KDResp NVMe/TCP PDU with the NQN contained in the Host NVMe Qualified Name (HOSTNQN) field in
the data portion of the Connect command to know that a CDC is performing a pull registration.
If the PLEO bit is cleared to ‘0’ and the ALLSUBE bit is set ‘1’ in the LSP field of the Get Log Page command,
then the DDC shall return all NVM subsystem discovery information entries in the resulting Discovery log
page sent to the CDC (i.e., the DDC shall not filter out any NVM subsystem discovery information entries).
If the PLEO bit is set to ‘1’ and the ALLSUBE bit is set ‘1’ in the LSP field of the Get Log Page command,
then the DDC shall return all NVM subsystem discovery information entries for NVM subsystem ports that
are presented through the same NVM subsystem p ort on the DDC that received the Get Log Page
command in the resulting Discovery log page sent to the CDC (i.e., the DDC shall not filter out any NVM
subsystem discovery information entries for NVM subsystem ports that are presented through the same
NVM subsystem port on the DDC that received the Get Log Page command).
An additional pull registration or a pull de-registration may be requested by either:
• sending a Discovery Log Page Change Asynchronous Event notification (Asynchronous Event
Information F0h) (refer to section 3.1.3.3.2); or
• requesting another pull registration.
If the CDC established an explicit persistent connection with the DDC when performing the previous pull
registration, then the DDC should request the additional pull registration or the pull de -registration by
sending a Discovery Log Page Change Asynchronous Event notification.
If the CDC did not establish an explicit persistent connection with the DDC when performing the previous
pull registration, then the DDC requests the additional pull registration or the pull de -registration by
requesting another pull registration.
Upon receiving a Discovery Log Page Change Asynchronous Event notification, the CDC performs the
additional pull registration or the pull de-registration by using the following sequence:
1. sending a Get Log Page command to the DDC with the LID field set to 70h to retrieve NVM
subsystem discovery information contained in the associated Discovery log page from that DDC.
The CDC may:
a. set the Port Local Entries Only (PLEO) bit to ‘1’ in the Log Specific Parameter (LSP) field in
that Get Log Page command to request that NVM subsystem discovery information entries for
only NVM subsystem ports that are presented through the same NVM subsys tem port on the
DDC that received the Get Log Page command be returned; and
b. set the All NVM Subsystem Entries (ALLSUBE) bit to ‘1’ in the LSP field in that Get Log Page
command to request that records for all NVM subsystem ports be returned;
and
