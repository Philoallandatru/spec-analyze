---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 637
headings: ["8.3.2.3 Fabric Zoning", "8.3.2.3.1 Model"]
---

# Source page 637

NVM Express® Base Specification, Revision 2.1

615
Figure 678: Kickstart Discovery Request and Response Sequence

 Fabric Zoning
8.3.2.3.1 Model
A Centralized Discovery controller (CDC) may provide centralized access control services (i.e., Fabric
Zoning) for an NVMe-oF IP-based fabric. CDC-based Fabric Zoning provides a way to control the Discovery
log pages provided in response to a Get Log Page command issued to the CDC. Fabric Zoning should be
based on a Zoning database maintained by the CDC and containing two fundamental data structures: the
ZoneDBConfig and the ZoneDBActive, as shown in Figure 679.
Figure 679: Zoning DB Abstract Representation
Zoning DB
ZoneDBConfig ZoneDBActive

The ZoneDBActive is a list of enforced ZoneGroups. The enforcement actions include:
• Discovery log page filtering;
• Discovery Log Page Change Asynchronous Event notifications (refer to section 8.3.2.4); and
• optionally, network level restrictions (refer to section 8.3.2.3.5.2).
The abstract representation of the ZoneDBActive is shown in Figure 680.
TCP SYN
TCP SYN-ACK
NVMe/TCP ICReq PDU
NVMe/TCP ICResp PDU
NVMe/TCP KDReq PDU
NVMe/TCP KDResp PDU
NVMe/TCP connection tear down
DDC CDC
