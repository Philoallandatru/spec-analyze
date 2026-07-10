---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 631
headings: ["8.3.1.4 CDC Operation", "8.3.1.4.1 CDC mDNS Initialization", "8.3.1.4.2 CDC DNS Initialization", "8.3.1.4.3 CDC Query", "8.3.1.4.4 CDC Processing of DNS-SD records", "8.3.1.4.5 CDC response to mDNS queries"]
---

# Source page 631

NVM Express® Base Specification, Revision 2.1

609
“_nvme-disc.<protocol>.local”
mDNS responses to queries for these service names shall contain the information described in section
8.3.1.1.2.
DDCs should ignore mDNS queries for the service name of “_ddcpull._sub._nvme -disc.<protocol>.local”
(refer to Figure 675), as the subtype “_ddcpull._sub” is obsolete.
 CDC Operation
8.3.1.4.1 CDC mDNS Initialization
During initialization (e.g., following a link transition or power cycle), before the CDC’s mDNS responder
function is enabled, the CDC shall probe to ensure the unique resource records the CDC are responsible
for are unique on the local link (refer to section 8.1 in RFC 6762).
Upon successful completion of the probe, the CDC shall announce (refer to section 8.2 in RFC 6762) its
newly registered resource records.
Upon announcing its resource records, the CDC’s mDNS responder function may be enabled and respond
to queries for the service name of “_cdc._sub._nvme -disc._<protocol>.local” as described in section
8.3.1.4.5.
A CDC should query for the presence of another CDC as defined in section 8.3.1.4.3 and process
responses as defined in section 8.3.1.4.4.
8.3.1.4.2 CDC DNS Initialization
During initialization (e.g., following a link transition or power cycle), a CDC may dynamically update DNS
records (refer to RFC 2136) by providing an update that includes the Resource Records defined in section
8.3.1.1.2.
8.3.1.4.3 CDC Query
A CDC may query for other CDC instances. When performed the mDNS or DNS query shall include a DNS
PTR record (refer to RFC 6763) with the name in the form of:
“_cdc._sub._nvme-disc.<protocol>.<domain.
The protocol field shall be set as shown in Figure 674.
The domain field shall be set as shown in Figure 676.
8.3.1.4.4 CDC Processing of DNS-SD records
Upon reception of an mDNS or DNS response that contains a DNS SRV record with the service name set
to “_cdc._sub._nvme-disc.<protocol>.local” the CDC may provide an alert to the administrator to indicate
the presence of more than one CDC in a broadcast domain.
A CDC should ignore an mDNS or DNS response that contains a DNS SRV record with the service name
set to “_ddcpull._sub._nvme-disc.<protocol>.local”.
8.3.1.4.5 CDC response to mDNS queries
A CDC may respond to mDNS queries for the service names of either:
“_nvme-disc.<protocol>.local”; or
“_cdc_sub._nvme-disc.<protocol>.local”.
mDNS responses to this query shall contain the information described in section 8.3.1.1.2.
