---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 632
headings: ["8.3.2 Centralized Discovery for IP-based Fabrics", "8.3.2.1 Overview", "8.3.2.1.1 Registration and Discovery Example"]
---

# Source page 632

NVM Express® Base Specification, Revision 2.1

610
 Centralized Discovery for IP-based Fabrics
 Overview
In configurations that consist of multiple NVM subsystems, the burden on administrators is reduced by
enabling hosts to automatically retrieve the list of NVM subsystem ports the host has been allowed to
access from a centralized location. This centralized  location is referred to as a Centralized Discovery
controller (CDC).
Hosts and NVM subsystems may become known to the CDC by explicitly registering their discovery
information as the result of a push registration (refer to section 8.3.2.2.1). If a host or Direct Discovery
controller (DDC) detects the presence of multiple CDCs, then that host or DDC should register their
discovery information with each CDC. Alternatively, the CDC may implicitly register discovery information
as a result of pr ocessing a Connect command from a host, or as a result of receiving a pull registration
request from a DDC (refer to section 8.3.2.2.2).
The CDC may filter the list of NVM subsystem ports returned in response to a Get Log Page command
from the host that requests the Discovery log page to include only the NVM subsystem ports that provide
access to namespaces allocated to that host. The process used to configure this filtering function is known
as Fabric Zoning (refer to section 8.3.2.3.
An overview of the registration and discovery process is described in section 8.3.2.1.1.
8.3.2.1.1 Registration and Discovery Example
The following example assumes the CDC’s effective Fabric Zoning configuration allows Host A to access
both NVM subsystem A and NVM subsystem B.
Figure 677: Registration and Discovery Example
