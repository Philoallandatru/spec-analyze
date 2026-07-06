---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 31
headings: ["1.5.36 Exported NVM Subsystem", "1.5.37 Exported Port", "1.5.38 Exported Port ID", "1.5.39 fabric (network fabric)", "1.5.40 Fabric Zoning", "1.5.41 firmware/boot partition image update command sequence", "1.5.42 firmware slot", "1.5.43 host", "1.5.44 host-accessible memory", "1.5.45 host management agent", "1.5.46 host memory", "1.5.47 idempotent command"]
---

# Source page 31

NVM Express® Base Specification, Revision 2.1

9
 Exported NVM Subsystem
A logical NVM subsystem that exports underlying NVM resources and that:
a) contains zero or more Exported Namespaces;
b) contains zero or more controllers;
c) contains zero or more Exported Ports; and
d) may contain an Allowed Host List.
 Exported Port
A port used to export an NVMe subsystem over a specific fabrics transport and represented by an Exported
Port ID.
 Exported Port ID
A port identifier used to specify an Exported Port.
 fabric (network fabric)
A network topology in which nodes pass data to each other.
 Fabric Zoning
A technique to specify access control configurations between hosts and NVM subsystems.
 firmware/boot partition image update command sequence
The sequence of one or more Firmware Image Download commands that download a firmware image or a
boot partition image followed by a Firmware Commit command that commits that downloaded image to a
firmware slot or a boot partition.
 firmware slot
A firmware slot is a location in a domain used to store a firmware image . The domain stores from one to
seven firmware images. Controllers in the same domain share the same firmware slots.
 host
An entity that interfaces to an NVM subsystem through one or more controllers and submits commands to
Submission Queues and retrieves command completions from Completion Queues.
 host-accessible memory
Memory that the host is able to access (e.g., host memory, Controller Memory Buffer (CMB), Persistent
Memory Region (PMR)).
 host management agent
A host management agent is a part of the host that provides an external management interface (e.g.,
Redfish) to external managers, typically via Admin commands to the controller (refer to the NVM Express
Management Interface Specification).
 host memory
Memory that may be read and written by both a host and a controller and that is not exposed by a controller
(i.e., Controller Memory Buffer or Persistent Memory Region). Host memory may be implemented inside or
outside a host (e.g., a memory region exposed by a device that is neither the host nor controller).
 idempotent command
A command that produces the same end state in the NVM subsystem and returns the same results if that
command is resubmitted one or more times with no intervening commands. Refer to section  9.6.3.1.
