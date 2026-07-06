---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 535
headings: ["8.1.9.3 Dispersed Namespace Access"]
---

# Source page 535

NVM Express® Base Specification, Revision 2.1

513
namespace identifier that uniquely identifies the dispersed namespace in all participating NVM subsystems
shall be either:
a) a Namespace Globally Unique Identifier (NGUID); or
b) a Universally Unique Identifier (UUID).
If the dispersed namespace has an NGUID and does not have a UUID, then:
a) that NGUID shall be the globally unique namespace identifier that uniquely identifies the dispersed
namespace in all participating NVM subsystems;
b) all participating NVM subsystems shall use the same NGUID value for the dispersed namespace;
and
c) the dispersed namespace may have an EUI64 that shall not be the globally unique namespace
identifier that uniquely identifies the dispersed namespace in all participating NVM subsystems.
If the dispersed namespace has a UUID and does not have an NGUID, then:
a) that UUID shall be the globally unique namespace identifier that uniquely identifies the dispersed
namespace in all participating NVM subsystems;
b) all participating NVM subsystems shall use the same UUID value for the dispersed namespace;
and
c) the dispersed namespace may have an EUI64 that shall not be the globally unique namespace
identifier that uniquely identifies the dispersed namespace in all participating NVM subsystems.
If the dispersed namespace has an NGUID and has a UUID, then:
a) the NGUID shall be the globally unique namespace identifier that uniquely identifies the dispersed
namespace in all participating NVM subsystems;
b) all participating NVM subsystems shall use the same NGUID value for the dispersed namespace;
and
c) the dispersed namespace may have an EUI64 that shall not be the globally unique namespace
identifier that uniquely identifies the dispersed namespace in all participating NVM subsystems.
 Dispersed Namespace Access
The host may discover if a namespace is capable of being accessed using controllers contained in two or
more participating NVM subsystems concurrently (i.e., may discover if a namespace may be a dispersed
namespace) by issuing an Identify command (i.e., CN S 00h if supported by the I/O Command Set or CNS
08h) to the controller for the specified NSID. If the namespace is capable of being accessed using
controllers contained in two or more participating NVM subsystem concurrently, then the controller shall set
the Dispersed Namespace (DISNS) bit to ‘1’ in the Namespace Multi -path I/O and Namespace Sharing
Capabilities (NMIC) field of the returned I/O Command Set Independent Identify Namespace data structure
(refer to Figure 319) or Identify Namespace data structure (refer to the NVM Command Set Specification ).
All controllers in each participating NVM subsystem that are able to provide access to a specific dispersed
namespace shall support the I/O Command Set associated with that dispersed namespace.
The host may discover the NQNs of participating NVM subsystems which contain controllers that are able
to provide access to a dispersed namespace by issuing a Get Log Page command for the Dispersed
Namespace Participating NVM Subsystems log page (i.e., the Log Page Identifier (LID) field set to 17h
(refer to section 5.1.12.1.23)).
Hosts specify support for dispersed namespaces by setting the Host Dispersed Namespace Support
(HDISNS) field to 1h in the Host Behavior Support feature by issuing a Set Features command with Feature
Identifier (FID) set to 16h (refer to Figure 407).
If the HDISNS field is set to 1h, the host specifies support for:
• treating all instances of a dispersed namespace as the same namespace, based upon the shared
globally unique namespace identifier (refer to section 8.1.9.2) when accessing that dispersed
namespace through multiple controllers on the same participating NVM subsystem;
