---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 662
headings: ["8.3.4.1 Fabric Secure Channel"]
---

# Source page 662

NVM Express® Base Specification, Revision 2.1

640
specific protocol that provides authentication, encryption, and integrity checking (e.g., IPsec; refer to RFC
4301 or TLS; refer to RFC 8446). NVMe in -band authentication is performed immediately after a Connect
command (refer to section 6.3) succeeds using the Authentication Send and Authentication Receive
commands (refer to section 6.1 and section 6.2) to tunnel authentication protocol commands between the
host and the controller.
Enrollment of the host and controller in an authentication mechanism, including provisioning of
authentication credentials to the host and controller, is outside the scope of this specification.
If both fabric secure channel and NVMe in -band authentication are used, the identities for these two
instances of authentication may differ for the same NVMe Transport connection. For example, if an iWARP
NVMe Transport is used with IPsec as the fabric sec ure channel technology, the IPsec identities for
authentication are associated with the IP network (e.g., DNS host name or IP address), whereas NVMe in-
band authentication uses NVMe identities (i.e., Host NQNs). The NVMe Transport binding specification
may provide further guidance and requirements on the relationship between these two identities, but
determination of which NVMe Transport identities are authorized to be used with which NVMe identities is
part of the security policy for the deployed NVM subsystem.
 Fabric Secure Channel
The Transport Requirements field in the Fabrics Discovery Log Page Entry (refer to Figure 294) indicates
whether a fabric secure channel shall be used for an NVMe Transport connection to an NVM subsystem.
The secure channel mechanism is specific to the type of fabric.
If establishment of a secure channel fails or a secure channel is not established when required by the
controller, the resulting errors are fabric -specific and may not be reported to the NVMe layer on the host.
Such errors may result in the controller bein g inaccessible to the host via the NVMe Transport connection
on which the failure to establish a fabric secure channel occurred.
An NVM subsystem that requires use of a fabric secure channel (i.e., as indicated by the TSC field in the
associated Discovery Log Page Entry) shall not allow capsules to be transferred until a secure channel has
been established for the NVMe Transport connection.
All Discovery Log Page Entries for an NVM subsystem should report the same value of TREQ to each host.
Discovery Log Page Entries for an NVM subsystem may report different values of TREQ to different hosts.
Figure 725 shows an example of secure channel establishment using TLS, the fabric secure channel
protocol for NVMe/TCP.
Figure 725: Example of TLS secure channel establishment
Connect Command
Connect Response
1. An NVMe/TCP-TLS transport session is
established
2. The Connect exchange is performed to set up an
NVMe Queue and associate host to controller
3. Secure channel and Queue are set up, ready
for subsequent operations
Host Controller
Secure channel and queue set up
NVMe/TCP-TLS TLS transport session establishment
