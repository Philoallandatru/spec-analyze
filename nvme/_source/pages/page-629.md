---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 629
headings: ["8.3.1.1.2.2 DNS SRV record", "8.3.1.1.2.3 DNS TXT record", "8.3.1.2 Host Operation", "8.3.1.2.1 Host Query"]
---

# Source page 629

NVM Express® Base Specification, Revision 2.1

607
The domain field shall be set as shown in Figure 676.
 DNS SRV record
The DNS SRV record provides the TCP port where the service instance can be reached and shall have a
name in the form of:
“<Instance>.<Service>.<Domain>”.
Instance (Instance name) is a vendor defined human readable string. Although use of a serial number is
discouraged in DNS -Based Service Discovery (refer to RFC 6763), an instance name that may be
meaningful to an IT administrator is a combination of the Mo del Number (MN), Serial Number (SN) and
Physical Fabric Interface (Physical Ports). For example:
“SubsystemIF-SerialNumber-SubsystemModel”.
The maximum length of the Instance Name field is 63 bytes (refer to RFC 6763).
The <Service> portion of the name (refer to section 4.1 of RFC 6763) can be further broken down into:
“<service name>.<protocol>”.
 DNS TXT record
The DNS TXT record provides additional information about the instance using key/value pairs in the form
of “key=value” separated by commas (refer to section 6.4 in RFC 6763).
For NVMe over Fabrics, the DNS TXT record shall include a key/value pair for protocol (p) and may include
a key/value pair describing an NQN.
DNS TXT record key/value pairs:
Protocol (p): the protocol field shall indicate the IP transport protocols that are supported by the
Discovery controller being advertised. For example:
“p=tcp”, “p=roce”, or “p=iwarp”.
NQN: the DNS TXT record may contain an nqn key/value pair. When it is included the NQN
provided shall be set to the unique NQN of the Discovery subsystem if one is available for use.
Otherwise, the well-known Discovery Service NQN (nqn.2014 -08.org.nvmexpress.discovery) may
be used.
As described in RFC 6763, the format of the data within a DNS TXT record is one or more strings, packed
together without any intervening gaps or padding bytes for word alignment.
The format of a TXT record that may be provided in a response is:
“<length byte>p=tcp<length byte>nqn=NQN.of.Discovery.subsystem”.
Using this format, an example of a TXT record that may be provided in a response is:
“05p=tcp1Enqn=NQN.of.Discovery.subsystem”.
 Host Operation
8.3.1.2.1 Host Query
As described in section 8.3.1.1.1, a host may transmit an mDNS or DNS query to discover CDC and DDC
instances that are present on the transport network. When used, the mDNS or DNS query shall include a
DNS PTR record (refer to RFC 6763) with the name in the form of:
“_nvme-disc.<protocol>.<domain>”.
The protocol field shall be set as shown in Figure 674.
The domain field shall be set as shown in Figure 676.
