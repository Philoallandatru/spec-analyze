---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 39
headings: ["2 Theory of Operation"]
---

# Source page 39

NVM Express® Base Specification, Revision 2.1

17
2 Theory of Operation
The NVM Express scalable interface is designed to address the needs of storage systems that utilize PCI
Express based solid state drives or fabric connected devices. The interface provides optimized command
submission and completion paths. It includes support for parallel operation by supporting up to 65,535 I/O
Queues with up to 65,535 outstanding commands per I/O Queue. Additionally, support has been added for
many Enterprise capabilities like end-to-end data protection (compatible with SCSI Protection Information,
commonly known as T10 DIF, and SNIA DIX standards), enhanced error reporting, and virtualization.
The interface has the following key attributes:
• Does not require uncacheable / MMIO register reads in the command submission or completion
path;
• A maximum of one MMIO register write or one 64B message is necessary in the command
submission path;
• Support for up to 65,535 I/O Queues, with each I/O Queue supporting up to 65,535 outstanding
commands;
• Priority associated with each I/O Queue with well-defined arbitration mechanism;
• All information to complete a 4  KiB read request is included in the 64B command itself, ensuring
efficient small I/O operation;
• Efficient and streamlined command set;
• Support for MSI/MSI-X and interrupt aggregation;
• Support for multiple namespaces;
• Efficient support for I/O virtualization architectures like SR-IOV;
• Robust error reporting and management capabilities; and
• Support for multi-path I/O and namespace sharing.
This specification defines a streamlined set of properties that are used to configure low level controller
attributes and obtain low level controller status. These properties have a transport specific mechanism for
defining access (e.g., memory-based transports use registers, whereas message-based transports use the
Property Get and Property Set commands ). The following are examples of functionality defined in
properties:
• Indication of controller capabilities;
• Status for controller failures (command status is provided in a CQE);
• Admin Queue configuration (I/O Queue configuration processed via Admin commands); and
• Doorbell registers for a scalable number of Submission and Completion Queues.
There are two defined models for communication between the host and the NVM subsystem, a memory-
based transport model and a message-based transport model. All NVM subsystems require the underlying
NVMe Transport to provide reliable NVMe command and data delivery. An NVMe Transport is an abstract
protocol layer independent of any physical interconnect properties. A taxonomy of NVMe Transports, along
with examples, is shown in Figure 4. An NVMe Transport may expose a memory-based transport model or
a message-based transport model. The message-based transport model has two subtypes: the message-
only transport model and the message/memory transport model.
A memory-based transport model is one in which commands, responses, and data are transferred between
a host and an NVM subsystem by performing explicit memory read and write operations (e.g., over PCIe).
A message-based transport model is one in which messages containing command capsules and response
capsules are sent between  a host and an NVM subsystem  (e.g., over a fabric). The two subtypes of
message-based transport models are differentiated by how data is sen t between a host and an NVM
subsystem. In the message-only transport model data is only sent between a host and an NVM subsystem
using capsules or messages. The message/memory transport model uses a combination of messages and
explicit memory read and write operations to transfer command capsules, response capsules and data
between a host and an NVM subsystem . Data may optionally be included in command capsules and
response capsules. Both the message-only transport model and the message/memory transport model are
