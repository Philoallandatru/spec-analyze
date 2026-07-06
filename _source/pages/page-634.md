---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 634
headings: ["8.3.2.2.1 Push Registrations and Push De-Registrations", "8.3.2.2.2 Pull Registrations and Pull De-Registrations"]
---

# Source page 634

NVM Express® Base Specification, Revision 2.1

612
Information from a Connect command (e.g., Host NQN) may be retained by a Discovery controller to provide
a limited set of discovery information. If the Discovery controller determines that the host that submitted the
Connect command is the same as a host f or which the Discovery controller has retained host discovery
information from a previous push registration, then the Discovery controller should continue to retain that
host discovery information.
A host:
• should use push registrations to register host discovery information with a CDC or DDC; and
• is not able to use pull registrations to register discovery information.
A CDC:
• may use push registrations to register host discovery information with a DDC; and
• shall not use pull registrations to register discovery information.
A DDC:
• should use push registrations when registering NVM subsystem discovery information with a CDC;
or
• may use pull registrations when registering NVM subsystem discovery information with a CDC.
Discovery information de -registration is the process of de -registering host or NVM subsystem discovery
information with a CDC or de-registering host discovery information with a DDC.
Discovery information de-registration may be performed using one of the following methods:
• a push de-registration (refer to section 8.3.2.2.1); or
• a pull de-registration (refer to section 8.3.2.2.2).
A host:
• may use push de-registrations to de-register host discovery information with a CDC or DDC; and
• is not able to use pull de-registrations to de-register discovery information.
A CDC:
• may use push de-registrations to de-register host discovery information with a DDC; and
• shall not use pull de-registrations to de-register discovery information.
A DDC:
• should use push de-registrations to de-register NVM subsystem discovery information with a CDC;
or
• may use pull de-registrations to de-register NVM subsystem discovery information with a CDC.
8.3.2.2.1 Push Registrations and Push De-Registrations
A push registration is performed using the Discovery Information Management command (refer to section
5.3.3) with the Task (TAS) field (refer to Figure 496) cleared to 0h to explicitly register discovery information
for a host or NVM subsystem with a Centralized Discovery controller (CDC) or explicitly register discovery
information for a host with a Direct Discovery controller (DDC).
A push de-registration is performed using the Discovery Information Management command with the TAS
field set to 1h to explicitly de -register discovery information for a host or NVM subsystem with a CDC or
explicitly de-register discovery information for a host with a DDC.
A DDC that performs push registrations or push de -registrations implements host functionality (e.g.,
submitting commands, processing command completions, providing a Host NQN, etc.).
8.3.2.2.2 Pull Registrations and Pull De-Registrations
A pull registration may be requested using a kickstart discovery request and response sequence (refer to
section 8.3.2.2.2).
