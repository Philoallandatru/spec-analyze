---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 139
headings: ["3.7 Resets", "3.7.1 NVM Subsystem Reset", "3.7.1.1 Single Domain NVM Subsystems", "3.7.1.2 Multiple Domain NVM Subsystems"]
---

# Source page 139

NVM Express® Base Specification, Revision 2.1

117
The initialization sequence (refer to section 3.5) should then be executed on that controller.
3.7 Resets
 NVM Subsystem Reset
Interactions between NVM Subsystem Reset and Power Loss Signaling processing are described in section
8.2.5.
 Single Domain NVM Subsystems
The scope of an NVM Subsystem Reset depends on whether the NVM subsystem supports multiple
domains. In an NVM subsystem that does not support multiple domains, the scope of the NVM Subsystem
Reset is the entire NVM subsystem.
An NVM Subsystem Reset is initiated when:
• Main power is applied to the NVM subsystem;
• A value of 4E564D65h (“NVMe”) is written to the NSSR.NSSRC field;
• Requested using a method defined in the NVM Express Management Interface Specification; or
• A vendor specific event occurs.
When an NVM Subsystem Reset occurs, the entire NVM subsystem is reset. This includes the initiation of
a Controller Level Reset on all controllers that make up the NVM subsystem, disabling of the Persistent
Memory Region associated with all controllers that make up the NVM subsystem, and any transport specific
actions defined in the applicable NVMe transport specification.
The occurrence of an NVM Subsystem Reset while power is applied to the NVM subsystem is reported by
the initial value of the CSTS.NSSRO field following the NVM Subsystem Reset. This field may be used by
host software to determine if the sudden loss of comm unication with a controller was due to an NVM
Subsystem Reset or some other condition.
The ability for host software to initiate an NVM Subsystem Reset by writing to the NSSR.NSSRC field is an
optional capability of a controller indicated by the state of the CAP.NSSRS field. An implementation may
protect the NVM subsystem from an inadvertent  NVM Subsystem Reset by not providing this capability to
one or more controllers that make up the NVM subsystem.
The occurrence of a vendor specific event that results in an NVM Subsystem Reset is intended to allow
implementations to recover from a severe NVM subsystem internal error that prevents continued normal
operation (e.g., fatal hardware or firmware error).
 Multiple Domain NVM Subsystems
The scope of an NVM Subsystem Reset depends on whether the NVM subsystem supports multiple
domains. In an NVM subsystem that supports multiple domains, the scope of the NVM Subsystem Reset
is either the controllers that are in a domain or the entire NVM subsystem.
An NVM Subsystem Reset on a domain is initiated when:
• Power is applied to that domain;
• A value of 4E564D65h (i.e., “NVMe”) is written to the NSSR.NSSRC field of one of the controllers
in that domain; or
• A vendor specific event occurs within that domain.
When an NVM Subsystem Reset occurs the entire domain is reset. This includes the initiation of a Controller
Level Reset on all controllers that are in the domain, disabling of the Persistent Memory Region associated
with all controllers that are in the domain, and any transport specific actions defined in the applicable NVMe
transport specification.
Alternatively, an NVM Subsystem Reset in an NVM subsystem that supports multiple domains may reset
the entire NVM subsystem.
