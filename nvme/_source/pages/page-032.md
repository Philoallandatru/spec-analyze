---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 32
headings: ["1.5.48 Identify Controller data structures", "1.5.49 Identify Namespace data structures", "1.5.50 I/O command", "1.5.51 I/O Completion Queue", "1.5.52 I/O controller", "1.5.53 I/O Submission Queue", "1.5.54 Media Unit", "1.5.55 memory-based controller", "1.5.56 message-based controller", "1.5.57 metadata", "1.5.58 MMC", "1.5.59 MMH"]
---

# Source page 32

NVM Express® Base Specification, Revision 2.1

10
 Identify Controller data structures
All controller data structures that are able to be retrieved via the Identify command:
• Identify Controller data structure (i.e., CNS 01h); and
• each of the I/O Command Set specific Identify Controller data structure (i.e., CNS 06h).
 Identify Namespace data structures
All namespace data structures that are able to be retrieved via the Identify command:
• Identify Namespace data structures (i.e., CNS 00h, CNS 09h, and CNS 11h);
• I/O Command Set Independent Identify Namespace data structures (i.e., CNS 08h and CNS 1Fh);
and
• I/O Command Set specific Identify Namespace data structures (i.e., CNS 05h, CNS 0Ah, and CNS
1Bh).
 I/O command
An I/O command is a command submitted to an I/O Submission Queue.
 I/O Completion Queue
An I/O Completion Queue is a Completion Queue that is used to indicate command completions and is
associated with one or more I/O Submission Queues.
 I/O controller
A controller that implements I/O queues and is intended to be used to access a non -volatile storage
medium.
 I/O Submission Queue
An I/O Submission Queue is a Submission Queue that is used to submit I/O commands for execution by
the controller (e.g., Read command and Write command for the NVM Command Set).
 Media Unit
A Media Unit represents a component of the underlying media in an NVM subsystem. Endurance Groups
are composed of Media Units.
 memory-based controller
A controller that supports a memory-based transport model (e.g., a PCIe implementation).
 message-based controller
A controller that supports a message-based transport model (e.g., a Fabrics implementation).
 metadata
Metadata is contextual information related to formatted user data (e.g., a particular LBA of data  as defined
in the NVM Command Set Specification ). The host may include metadata to be stored by the NVM
subsystem if storage space is provided by the controller. Refer to the applicable I/O Command Set
specification for details.
 MMC
A Migration Management Controller. Refer to section 8.1.12.
 MMH
A Migration Management Host. Refer to section 8.1.12.
