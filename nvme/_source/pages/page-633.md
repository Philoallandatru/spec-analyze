---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 633
headings: ["8.3.2.2 Discovery Information Registration and De-Registration"]
---

# Source page 633

NVM Express® Base Specification, Revision 2.1

611
Each of the numbered steps in Figure 677 are described below:
1. A fabric interface on NVM subsystem A is registered with the CDC. This is accomplished by either:
a. using the Discovery Information Management command (refer to section 5.3.3) to perform a
push registration (refer to section 8.3.2.2.1);
b. notifying the CDC that a pull registration (refer to section 8.3.2.2.2) is required and the NVM
subsystem port must be implicitly registered by the CDC; or
c. administrative configuration (e.g., the NVM subsystem port was manually configured on the
CDC).
2. Host A should establish an explicit persistent connection with the CDC. The method that a host
uses to obtain the information necessary to connect to the CDC via the NVMe Transport may be:
a. implementation specific;
b. fabric specific;
c. known in advance (e.g., a well-known address);
d. administratively configured; or
e. for IP based fabrics, Automated Discovery of Discovery Controllers for IP Based Fabrics (refer
to section 8.3.1) may be used.
3. Host A uses the Discovery Information Management command to perform a push registration and
explicitly register its discovery information with the CDC.
4. Host A should send one or more Asynchronous Event Request commands to the CDC in order to
be notified about any changes that occur to the Discovery log page.
5. Host A uses the Get Log Page command to retrieve the Discovery log page from the CDC.
6. The CDC responds to the Get Log Page command with a Discovery log page containing one
Discovery Log Page Entry for the interface on NVM subsystem A.
7. If the SUBTYPE field of the Discovery Log Page Entry for NVM subsystem A returned from the
CDC is set to 02h (i.e., NVM subsystem), then Host A should connect to the I/O controller on NVM
subsystem A.
8. A fabric interface on NVM subsystem B is registered with the CDC. This is accomplished by either:
a. using the Discovery Information Management command to perform a push registration;
b. notifying the CDC that a pull registration is required and the NVM subsystem port must be
implicitly registered by the CDC; or
c. administratively configured (e.g., the NVM subsystem port was manually configured on the
CDC).
9. The CDC sends a Discovery Log Page Change Asynchronous Event notification (Asynchronous
Event Information F0h) to notify Host A that the Discovery log page has changed.
10. Host A uses the Get Log Page command to retrieve the Discovery log page from the CDC.
11. The CDC responds to the Get Log Page command with a Discovery log page containing two
Discovery Log Page Entries: one for the interface on NVM subsystem A and another for the
interface on NVM subsystem B.
12. If the SUBTYPE field of the Discovery log page for NVM subsystem B returned from the CDC is
set to 02h (i.e., NVM subsystem), then Host A should connect to the I/O controller on NVM
subsystem B. The connection between Host A and NVM subsystem A remains unc hanged.
 Discovery Information Registration and De-Registration
Discovery information registration is the process of registering host or NVM subsystem discovery
information with a Centralized Discovery controller (CDC) or registering host discovery information with a
Direct Discovery controller (DDC).
Discovery information registration may be performed:
• using a push registration (refer to section 8.3.2.2.1); or
• using a pull registration (refer to section 8.3.2.2.21).
