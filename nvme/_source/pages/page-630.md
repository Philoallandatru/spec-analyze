---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 630
headings: ["8.3.1.2.2 Host Processing of DNS-SD Records", "8.3.1.3 DDC Operation", "8.3.1.3.1 DDC mDNS Initialization", "8.3.1.3.2 DDC DNS Initialization", "8.3.1.3.3 DDC Query", "8.3.1.3.4 DDC Processing of DNS-SD records", "8.3.1.3.5 DDC response to mDNS queries"]
---

# Source page 630

NVM Express® Base Specification, Revision 2.1

608
8.3.1.2.2 Host Processing of DNS-SD Records
Upon reception of an mDNS or DNS response that contains a DNS SRV record with the service name set
to “_nvme-disc.<protocol>.local”. The host interface may use the IP address in the A or AAAA record as
the destination IP address for a subsequent Fabrics Co nnect command and attempt to perform Discovery
Information Registration (refer to section 8.3.2.2).
 DDC Operation
8.3.1.3.1 DDC mDNS Initialization
During initialization (e.g., following a link transition or power cycle), before the DDC’s mDNS responder
function is enabled, the DDC shall probe to ensure the unique resource records the DDC are responsible
for are unique on the local link (refer to section 8.1 in RFC 6762).
Upon successful completion of the probe, the DDC shall Announce (refer to section 8.2 in RFC 6762) its
newly registered resource records.
Upon announcing its resource records, if a DDC:
a. has not been configured to perform push registration (refer to section 8.3.2.2.1), or has not been
configured to request a pull registration (refer to section 8.3.2.2.2) from a CDC, it may respond to
mDNS queries for the service name of “_nvme-disc.<protocol>.local”; or
b. has been configured to perform push registration (refer to section 8.3.2.2.1) with a CDC, it should
not respond to mDNS queries for the service name of “_nvme -disc.<protocol>.local”, unless it has
been administratively configured to do so or until it has performed a query and determined a CDC
is not present as defined in section 8.3.1.3.3.
8.3.1.3.2 DDC DNS Initialization
During initialization (e.g., following a link transition or power cycle), a DDC may dynamically update DNS
records (refer to RFC 2136) by providing an update that includes the Resource Records defined in section
8.3.1.1.2.
8.3.1.3.3 DDC Query
A DDC may determine if a CDC is present by transmitting a query that includes a DNS PTR record (refer
to RFC 6763) with the name in the form of:
“_cdc._sub._nvme-disc.<protocol>.<domain>”.
The protocol field shall be set as shown in Figure 674.
The domain field shall be set as shown in Figure 676.
8.3.1.3.4 DDC Processing of DNS-SD records
Upon reception of an mDNS or DNS response that contains a DNS SRV record with the service name set
to “_cdc._sub._nvme-disc”, the DDC may use the IP address in the A or AAAA record as the destination IP
address to either:
a. Perform push registration (refer to section 8.3.2.2.1) with the CDC; or
b. Request a pull registration (refer to section 8.3.2.2.2) from the CDC (e.g., using Kickstart Discovery
Request PDU (KDReq) section in the NVMe Transport Specification).
If a DDC supports mDNS and has been configured to perform push registration (refer to section 8.3.2.2.1),
or has been configured to request a pull registration (refer to section 8.3.2.2.2) from a CDC, the DDC should
cease responding to mDNS requests for the service name of “_nvme -disc.<protocol>.local” if a CDC is
detected as defined in section 8.3.1.3.3.
8.3.1.3.5 DDC response to mDNS queries
A DDC may respond to mDNS queries for the service names of:
