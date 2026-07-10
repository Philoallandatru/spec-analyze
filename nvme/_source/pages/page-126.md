---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 126
headings: []
---

# Source page 126

NVM Express® Base Specification, Revision 2.1

104
Figure 82: Queue Creation Flow
ADMIN OR I/O QUEUE CREATED
HOST CONTROLLER

Controller ID and
AUTHREQ returned
If AUTHREQ   0

The controller initialization steps after an association is established are described below. For determining
capabilities or configuring properties, the host uses the Property Get command and Property Set command,
respectively.
1. NVMe in-band authentication is performed if required (refer to section 8.3.4.2);
2. The host determines the controller capabilities;
3. The host determines the supported I/O Command Sets by checking the state of CAP.CSS and
appropriately initializing CC.CSS as follows:
a. If the CAP.CSS.NOIOCSS bit is set to ‘1’, then the CC.CSS field should be set to 111b;
b. If the CAP.CSS.IOCSS bit is set to ‘1’, then the CC.CSS field should be set to 110b; and
c. If the CAP.CSS.IOCSS bit is cleared to ‘0’ and the CAP.CSS.NCSS bit is set to ‘1’, then the
CC.CSS field should be set to 000b;
4. The host configures controller settings. Specific settings include:
a. The arbitration mechanism should be selected in CC.AMS; and
b. The memory page size should be initialized in CC.MPS;
5. The controller should be enabled by setting CC.EN to ‘1’;
6. The host should wait for the controller to indicate the controller is ready to process commands. The
controller is ready to process commands when CSTS.RDY is set to ‘1’;
7. The host determines the configuration of the controller by issuing the Identify command specifying
the Identify Controller data structure (i.e., CNS 01h);
8. The host determines any I/O Command Set specific configuration information as follows:
a. If the CAP.CSS.IOCSS bit is set to ‘1’, then the host does the following:
i. Issue the Identify command specifying the Identify I/O Command Set data structure (CNS
1Ch); and
