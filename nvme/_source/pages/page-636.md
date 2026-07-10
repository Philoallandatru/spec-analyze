---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 636
headings: ["8.3.2.2.2.1 Kickstart Discovery Pull Registration Requests"]
---

# Source page 636

NVM Express® Base Specification, Revision 2.1

614
2. replacing all existing NVM subsystem discovery information from the previous pull registration with
the newly retrieved NVM subsystem discovery information (i.e., only the most recent NVM
subsystem discovery information from that DDC is retained by the CDC).
Upon receiving another pull registration request, the CDC performs the additional pull registration or the
pull de-registration by using the following sequence:
1. acknowledging that pull registration request (refer to section 8.3.2.2.2.1 for kickstart discovery);
and
2. sending a Connect command to the DDC to establish an NVMe connection with that DDC ; and
3. sending a Get Log Page command to the DDC with the LID field set to 70h to retrieve NVM
subsystem discovery information contained in the associated Discovery log page from that DDC.
The CDC may:
a. set the Port Local Entries Only (PLEO) bit to ‘1’ in the Log Specific Parameter (LSP) field of the
Get Log Page command to request that NVM subsystem discovery information entries for only
NVM subsystem ports that are presented through the same NVM subsyst em port on the DDC
that received the Get Log Page command be returned; and
b. set the All NVM Subsystem Entries (ALLSUBE) bit to ‘1’ in the LSP field of the Get Log Page
command to request that records for all NVM subsystem ports be returned;
and
4. replacing all existing NVM subsystem discovery information from the previous pull registration with
the newly retrieved NVM subsystem discovery information (i.e., only the most recent NVM
subsystem discovery information from that DDC is retained by the CDC).
 Kickstart Discovery Pull Registration Requests
Figure 678 illustrates the process used during a kickstart discovery request and response sequence. The
first step is to establish a TCP connection between a Direct Discovery controller (DDC) and a Centralized
Discovery controller (CDC). For kickstart discovery, the CDC acts as the passive side of the TCP connection
and is set to “listen” for DDC-initiated TCP connection establishment requests. The IP address used by the
DDC to establish the TCP connection with the CDC may be obtained from the A or AAAA record provid ed
in an mDNS response from the CDC, as described in section 8.3.1.3.4.
Once a TCP connection has been established, if an ICReq NVMe/TCP PDU (refer to the Initialize
Connection Request PDU section in the NVMe TCP Transport Specification) from a DDC with the Kickstart
Discovery Connection (KDCONN) bit set to ‘1’ in the FLAGS field is received by a CDC, then the CDC shall
respond with an ICResp NVMe/TCP PDU (refer to the Initialize Connection Response PDU section in the
NVMe TCP Transport Specification) to establish a kickstart discovery NVMe/TCP connect ion.
Once a kickstart discovery NVMe/TCP connection has been established, if a KDReq NVMe/TCP PDU (refer
to the Kickstart Discovery Request PDU section in the NVMe TCP Transport Specification) from a DDC is
received by a CDC, then the CDC shall respond with a K DResp NVMe/TCP PDU (refer to the Kickstart
Discovery Response PDU section in the NVMe TCP Transport Specification) to accept the pull registration
request and connection parameters. The KDResp NVMe/TCP PDU sent by the CDC shall include the NQN
of that CDC in the CDC NVMe Qualified Name (CDCNQN) field. After sending a KDResp NVMe/TCP PDU
to the DDC where the Kickstart Status (KSSTAT) field is set to SUCCESS, the kickstart discovery
NVMe/TCP connection should be torn down, and the CDC performs a pull registra tion with the DDC as
described in section 8.3.2.2.2 using the connection parameters obtained from the KDReq NVMe/TCP PDU.
If the total number of discovery information entries being registered by the pull registration (i.e., as specified
by the Number of Discovery Information Entries (NUMDIE) field in the KDReq NVMe/TCP PDU) exceeds
the available capacity for new discovery information entries on the CDC, then the CDC shall send a KDResp
NVMe/TCP PDU to the DDC where the KSSTAT field is set to FAILURE and the Failure Reason (FAILRSN)
field is set to Insufficient Discovery Resources.
