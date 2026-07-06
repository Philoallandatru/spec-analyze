---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 40
headings: []
---

# Source page 40

NVM Express® Base Specification, Revision 2.1

18
referenced as message -based transport models throughout this specification when the description is
applicable to both subtypes.
An NVM subsystem is made up of a single domain or multiple domains as described in section 3.2.5. An
NVM subsystem may optionally include a non -volatile storage medium, and an interface between the
controller(s) of the NVM subsystem and the non -volatile storage medium. Controllers expose this non -
volatile storage medium to hosts through namespaces. An  NVM subsystem is not required to have the
same namespaces attached to all controllers. An NVM subsystem that supports a Discovery controller does
not support any other controller type . A Discovery Service is an NVM subsystem that supports Discovery
controllers only (refer to section 3.1).
Figure 4: Taxonomy of Transport Models
NVMe Transport Models
Memory
Commands/Responses & Data
use Shared Memory
Message
Commands/Responses use Capsules
Data may use Capsules or Messages
Message / Memory
Commands/Responses use Capsules
Data may use
Capsules or Shared Memory
Example Transport
PCI Express
Example Transports
Fibre Channel
TCP
Example Transports
RDMA
(InfiniBand, RoCE, iWARP)

The capabilities and settings that apply to an NVM Express controller are indicated in the Controller
Capabilities (CAP) property and the Identify Controller data structure (refer to Figure 312).
A namespace is a set of resources (e.g., formatted non -volatile storage) that may be accessed by a host.
A namespace has an associated namespace identifier that a host uses to access that namespace. The set
of resources may consist of non-volatile storage and/or other resources.
Associated with each namespace is an I/O Command Set that operates on that namespace.  An NVM
Express controller may support multiple namespaces. Namespaces may be created and deleted using the
Namespace Management command and Capacity Management command. The Identify Namespace data
structures (refer to section 1.5.49) indicate capabilities and settings that are specific to a particular
namespace.
The NVM Express interface is based on a paired Submission and Completion Queue mechanism.
Commands are placed by host software into a Submission Queue. Completions are placed into the
associated Completion Queue by the controller.
There are three types of commands that are defined in NVM Express: Admin commands, I/O commands,
and Fabrics commands. Figure 5 shows these different command types.
