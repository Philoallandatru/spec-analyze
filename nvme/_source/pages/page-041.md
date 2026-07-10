---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 41
headings: ["2.1 Memory-Based Transport Model (PCIe)"]
---

# Source page 41

NVM Express® Base Specification, Revision 2.1

19
Figure 5: Types of NVMe Command Sets
NVMe Command Sets
Fabrics
Command Set
Admin
Command Set
I/O
Command Sets
NVM
Command
Set
Zoned
Namespace
Command
Set
Key Value
Command
Set

An Admin Submission  Queue and associated Completion Queue exist for the purpose of controller
management and control (e.g., creation and deletion of I/O Submission and Completion Queues, aborting
commands, etc.). Only commands that are part of the Admin Command Set or the Fabrics Command Set
may be submitted to the Admin Submission Queue.
An I/O Command Set is used with an I/O queue pair. This specification defines common I/O commands .
I/O Command Sets are defined in I/O Command Set specifications. The example I/O Command Sets shown
in Figure 5 are the NVM Command Set, the Key Value Command Set, and the Zoned Namespace
Command Set. Other I/O Command Sets include the Computational Programs Command Set and the SLM
Command Set.
The Fabrics Command Set is NVMe over Fabrics specific. Fabrics Command Set commands are used for
operations specific to NVMe over Fabrics including establishing a connection, NVMe in -band
authentication, and to get or set a property. All Fabrics commands m ay be submitted on the Admin
Submission Queue and some Fabrics commands may also be submitted on an I/O  Submission Queue.
Unlike Admin and I/O commands, Fabrics commands are processed by a controller regardless of whether
the controller is enabled (i.e., regardless of the state of CC.EN).
2.1 Memory-Based Transport Model (PCIe)
In the memory-based model, Submission and Completion Queues are allocated in memory.
Host software creates queues, up to the maximum supported by the controller. Typically, the number of
command queues created is based on the system configuration and anticipated workload. For example, on
a four core processor based system, there may be a queue pair per core to avoid locking and ensure data
structures are created in the appro priate processor core’s cache. Figure 6 provides a graphical
representation of the queue pair mechanism, showing a 1:1 mapping between Submission Queues and
Completion Queues. Figure 7 shows an example where multiple I/O Submission Queues utilize the same
I/O Completion Queue on Core B. Figure 6 and Figure 7 show that there is always a 1:1 mapping between
the Admin Submission Queue and Admin Completion Queue.
