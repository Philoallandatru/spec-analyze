---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 43
headings: ["2.2 Message-Based Transport Model (Fabrics)", "2.2.1 Fabrics and Transports"]
---

# Source page 43

NVM Express® Base Specification, Revision 2.1

21
CQ slot. A Phase Tag (P) bit is defined in the completion queue entry to indicate whether an entry has been
newly posted without the host consulting a register (refer to section 4.2.4). The Phase Tag bit enables the
host to determine whether entries are new or not.
2.2 Message-Based Transport Model (Fabrics)
The message-based transport model used for NVMe over Fabrics has the following differences from the
memory-based transport model:
• There is a one -to-one mapping between I/O Submission Queues and I/O Completion Queues.
NVMe over Fabrics does not support multiple I/O Submission Queues being mapped to a single
I/O Completion Queue;
• NVMe over Fabrics does not define an interrupt mechanism that allows a controller to generate a
host interrupt. It is the responsibility of the host fabric interface (e.g., Host Bus Adapter) to generate
host interrupts;
• NVMe over Fabrics uses different mechanism s for I/O Submission Queue and I/O Completion
Queue creation and deletion (refer to section 3.5);
• NVMe over Fabrics does not support transferring metadata from a separate buffer  (e.g., does not
support the Metadata Pointer field, refer to Figure 92);
• NVMe over Fabrics does not support PRPs but requires use of SGLs for Admin, I/O, and Fabrics
commands. This differs from the memory-based transport model where SGLs are not supported
for Admin commands and are optional for I/O commands;
• NVMe over Fabrics does not support Completion Queue flow control  (refer to section  3.3.1.2.1).
This requires that the host ensures there are available Completion Queue slots before submitting
new commands; and
• NVMe over Fabrics allows Submission Queue flow control to be disabled if the host and controller
agree to disable Submission Queue flow control. If Submission Queue flow control is disabled, the
host is required to ensure that there are available Submission Queue slots before submitting new
commands.
 Fabrics and Transports
NVMe over Fabrics utilizes the protocol layering shown in Figure 8. This specification defines core aspects
of the architecture that are independent of the NVMe Transport. An NVMe Transport binding specification
is used to describe any NVMe Transport specific specialization as well as how the services required by the
NVMe interface are mapped onto the corresponding NVMe Transport. The native fabric communication
services and other functionality used by the NVMe interface and NVMe Transports (e.g., the Fabric Protocol
and Fabric Physical layers in Figure 8) are outside the scope of the NVMe family of specifications.
