---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 627
headings: ["8.3.1.1 Discovery of NVMe-oF Discovery Controllers", "8.3.1.1.1 Query"]
---

# Source page 627

NVM Express® Base Specification, Revision 2.1

605
DNS-SD information may be retrieved using mDNS (refer to RFC 6762) or from a DNS server (refer to RFC
1034 and RFC 1035).
When DNS-SD is used as described in this section, hosts, CDCs and DDCs may use DNS -SD to perform
automated discovery of Discovery controllers in IP fabrics consisting of:
a) Two IP interfaces (i.e., a host and an NVM subsystem) physically connected to one another (refer
to Figure 671);
b) Many IP interfaces participating in one or more Broadcast Domains (refer to Figure 672); or
c) A Centralized Discovery controller and many IP interfaces that may reside in multiple Broadcast
Domains (refer to Figure 673).
mDNS is a multicast protocol that allows discovery of IP interfaces within the same Broadcast Domain.
Discovering IP interfaces outside of the Broadcast Domain using mDNS requires either the use of RFC
8766 or an mDNS NVMe-oF proxy. An mDNS NVMe-oF proxy is an mDNS responder that is responsive to
queries for either the “_nvme -disc” service or the “_cdc._sub._nvme -disc” service and responds with the
information defined in section 8.3.1.1.2.
Figure 671: Configuration A - Two IP Interfaces

Figure 672: Configuration B – Multiple IP Interfaces without a CDC

Figure 673: Configuration C – Multiple IP interfaces with a CDC

 Discovery of NVMe-oF Discovery Controllers
8.3.1.1.1 Query
To facilitate the discovery of Discovery controller IP fabric interface addresses, hosts, CDCs and DDCs
may transmit an mDNS query (refer to RFC 6762) or a DNS query (refer to RFC 1034 and RFC 1035) that
includes a DNS PTR record (refer to RFC 6763) with the name in the form of:
“<Service>.<Domain>”.
The <Service> portion of the name can be further broken down into:
1 Host 1
(h1)
Subsystem 1
(s1)A DDC
2
Host 1
(h1)
Host n
(hn)
Subsystem 1
(s1)
IP fabric
Subsystem n
(sn)
B
DDC
DDC
2
Host 1
(h1)
Host n
(hn)
Subsystem 1
(s1)
IP fabric
Subsystem n
(sn)
C
DDC
DDCCDC
