---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 24
headings: ["1.2 Scope", "1.3 Outside of Scope", "1.4 Conventions", "1.4.1 Keywords", "1.4.1.1 mandatory", "1.4.1.2 may", "1.4.1.3 obsolete"]
---

# Source page 24

NVM Express® Base Specification, Revision 2.1

2
1.2 Scope
This specification defines a set of properties  and commands that comprise the interface required for
communication with a controller in an NVM subsystem. These properties are to be implemented by a n
instance of a controller using a specific NVMe Transport. This specification also defines common aspects
of the NVMe I/O Command Sets that may be supported by a controller.
There are three types of controllers with different capabilities (refer to section 3.1.3):
a) I/O controllers;
b) Discovery controllers; and
c) Administrative controllers.
In this document the generic term controller is often used instead of enumerating specific controller types
when applicable controller types may be determined from the context.
1.3 Outside of Scope
The property interface and command set are specified apart from any usage model for the NVM, but rather
only specifies the communication interface to the NVM subsystem. Thus, this specification does not specify
whether the NVM subsystem is used as a solid-state drive, a main memory, a cache memory,  a backup
memory, a redundant memory, etc.  Specific usage models are outside the scope, optional, and not
licensed.
This specification defines requirements and behaviors that are implementation agnostic. The
implementation of these requirements and behaviors are outside the scope of this specification. For
example, an NVM subsystem that follows this specification may be  implemented by an SSD that attaches
directly to a fabric, a device that translates between a fabric and a PCIe NVMe SSD, or software running
on a general-purpose server.
This interface is specified above any non-volatile media management, like wear leveling. Erases and other
management tasks for NVM technologies like NAND are abstracted.
This specification does not contain any information on caching algorithms or techniques.
The implementation or use of other published specifications referred to in this specification, even if required
for compliance with the specification, are outside the scope of this specification ( e.g., PCI, PCI Express ,
and PCI-X). This includes published specifications for fabrics and other technologies referred to by this
document or any NVMe Transport binding specification.
1.4 Conventions
 Keywords
Several keywords are used to differentiate between different levels of requirements.
 mandatory
A keyword indicating items to be implemented as defined by this specification.
 may
A keyword that indicates flexibility of choice with no implied preference.
 obsolete
A keyword indicating functionality that was defined in a previous version of the NVM Express specification
and that has been removed from this specification.
