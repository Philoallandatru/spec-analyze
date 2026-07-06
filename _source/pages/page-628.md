---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 628
headings: ["8.3.1.1.2 Response", "8.3.1.1.2.1 DNS PTR record"]
---

# Source page 628

NVM Express® Base Specification, Revision 2.1

606
“<service name>.<protocol>”.
For NVMe over Fabrics, the DNS PTR record included in the mDNS or DNS query shall be in the form of:
“_nvme-disc.<protocol>.<domain>”; or
“_<subtype>._sub._nvme-disc.<protocol>.<domain>”.
The protocol field shall be set as shown in Figure 674.
The subtype field shall be set as shown in Figure 675.
The domain field shall be set as shown in Figure 676.
Figure 674: mDNS Protocol Field
NVMe-oF Transport Fabric Protocol mDNS <protocol> Field
TCP TCP “_tcp”
RDMA RoCE “_udp”
RDMA iWARP “_tcp”

Figure 675: mDNS Subtype
Subtype Usage
“_cdc” Used by CDC and DDC instances to detect the presence of a CDC service.
“_ddcpull” Obsolete.

Figure 676: mDNS Domain
Domain Usage
“local” Used for mDNS
<FQDN> A fully qualified domain name may be used when DNS -SD information is retrieved from a DNS
server.
8.3.1.1.2 Response
The responses that may be received for an mDNS or DNS query include the following records as described
in DNS-Based Service Discovery (refer to RFC 6763):
• A DNS PTR record (refer to section 3.3.12 in RFC 1035);
• a DNS SRV record (refer to RFC 2782);
• a DNS TXT record (refer to section 3.3.14 in RFC 1035); and
• an A record and/or AAAA record providing the IPv4 and IPv6 IP addresses respectively.
 DNS PTR record
The DNS PTR record included in the mDNS or DNS response shall be in the form of:
“<Service>.<Domain>”.
The <Service> portion of the name can be further broken down into:
“<service name>.<protocol>”.
For NVMe over Fabrics, the DNS PTR record included in the mDNS or DNS response shall be in the form
of:
“_nvme-disc.<protocol>.<domain>”; or
“_<subtype>._sub._nvme-disc.<protocol>.<domain>”
The protocol field shall be set as shown in Figure 674.
The subtype field shall be set as shown in Figure 675.
