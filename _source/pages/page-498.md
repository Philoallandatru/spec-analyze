---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 498
headings: []
---

# Source page 498

NVM Express® Base Specification, Revision 2.1

476
• set the Report ANA Non-Optimized State (RANANOS) bit to ‘1’ in the ANACAP field in the Identify
Controller data structure if ANA Non-Optimized state is able to be reported;
• set the Report ANA Inaccessible State (RANAIS) bit  to ‘1’ in the ANACAP field in the Identify
Controller data structure if ANA Inaccessible state is able to be reported;
• set the Report ANA Persistent Loss State (RANAPLS) bit to ‘1’ in the ANACAP field in the Identify
Controller data structure if ANA Persistent Loss state is able to be reported;
• set the Report ANA Change State (RANACS) bit to ‘1’ in the ANACAP field in the Identify Controller
data structure if ANA Change state is able to be reported;
• support Asymmetric Namespace Access Change Notices (refer to section 5.1.25.1.5); and
• support the Asymmetric Namespace Access log page (refer to section 5.1.12.1.13).
Namespaces attached to a controller that supports Asymmetric Namespace Access Reporting shall:
• be members of an ANA Group; and
• supply a valid ANA Group Identifier in the ANA Group Identifier (ANAGRPID) field in the Identify
Namespace data structure (refer to the applicable I/O Command Set specification).
A controller that supports Asymmetric Namespace Access Reporting may also support multiple domains
(refer to section 3.2.5).
Figure 585 shows an example of an NVM subsystem where access characteristics vary as a result of the
presence of two independent domains. In this example, the non -volatile storage media for namespace B
and for namespace C are contained within the same domain that contains controller 2. As a result, controller
2 provides optimized access to namespace B and to namespace C while controller 1 does not provide
optimized access to namespace B or to namespace C.  In an NVM subsystem that supports multiple
domains (refer to section 3.2.5), the Media Access Boundary shown in Figure 585 may be a Communication
boundary as shown in Figure 71 and Figure 72.
Figure 585: Namespace B and C optimized through Controller 2
NVMe Controller 1
NS
B
NSID 2
NVMe Controller 2
Port x Port yNVM
Subsystem
NSID 3NSID 3
NS
C
NSID 2
Media Access
Boundary

To provide optimized access to namespace B through controller 1, the NVM subsystem may be
administratively reconfigured, or may perform autonomous internal reconfiguration actions that change the
access characteristics of namespace B when accessed through controller 1 and controller 2 as shown in
Figure 586. Controller 2 provides optimized access to namespace C while controller 1 provides optimized
access to namespace B. In an NVM subsystem that supports multiple domains (refer to section 3.2.5), the
Media Access Boundary shown in Figure 586 may be a Communication boundary as shown in Figure 71
and Figure 72.
