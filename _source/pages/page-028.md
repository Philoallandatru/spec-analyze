---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 28
headings: ["1.5.3 admin label UTF-8", "1.5.4 Admin Queue", "1.5.5 Administrative controller", "1.5.6 allocated namespace", "1.5.7 Allowed Host List", "1.5.8 arbitration burst", "1.5.9 arbitration mechanism", "1.5.10 association", "1.5.11 audit", "1.5.12 authentication commands", "1.5.13 cache", "1.5.14 candidate command"]
---

# Source page 28

NVM Express® Base Specification, Revision 2.1

6
 admin label UTF-8
A UTF-8 string. Refer to section 1.4.2 for UTF-8 string requirements. Refer to section 1.5.1 for admin label
usage.
 Admin Queue
The Admin Queue is the Submission Queue and Completion Queue with identifier 0.  The Admin
Submission Queue and corresponding Admin Completion Queue are used to submit administrative
commands and receive completions for those administrative commands, respectively.
The Admin Submission Queue is uniquely associated with the Admin Completion Queue.
 Administrative controller
A controller that exposes capabilities that allow a host to manage an NVM subsystem. An Administrative
controller does not implement I/O Queues, provide access to data or metadata associated with user data
on a non -volatile storage medium, or support namespaces attached to the Administrative controller (i.e.,
there are never any active NSIDs).
 allocated namespace
A namespace that is associated with an allocated NSID.
 Allowed Host List
A list of hosts (identified by Host NQN and Host Identifier) present in each Exported NVM Subsystem that
are granted access to the Exported NVM Subsystem via an Exported Port.
 arbitration burst
The maximum number of commands that may be fetched by an arbitration mechanism at one time from a
Submission Queue.
 arbitration mechanism
The method used to determine which Submission Queue is selected next to fetch commands for execution
by the controller. Refer to section 3.4.4.
 association
An exclusive communication relationship between a particular controller and a particular host that
encompasses the Admin Queue and all I/O Queues of that controller.
 audit
The process of accessing media to determine correct operation of a sanitize operation. Refer to section
8.1.24 and to ISO/IEC 27040.
 authentication commands
Used to refer to Fabrics Authentication Send or Authentication Receive commands.
 cache
A data storage area used by the NVM subsystem, that is not accessible to a host, and that may contain a
subset of user data stored in the non-volatile storage media or may contain user data that is not committed
to non-volatile storage media.
 candidate command
A candidate command is a submitted command  which has been transferred into the controller and  the
controller deems ready for processing.
