---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 108
headings: ["3.3.1.2.1 Completion Queue Flow Control"]
---

# Source page 108

NVM Express® Base Specification, Revision 2.1

86
Note: The consumer shall take queue wrap conditions into account.
Creation and deletion of memory-based transport Submission Queue and associated Completion Queues
are required to be ordered correctly by host software. Host software creates the Completion Queue before
creating any associated Submission Queue. Submission Queues may be created at any time after the
associated Completion Queue is created. Host software deletes all associated Submission Queues prior to
deleting a Completion Queue. To abort all commands submitted to the Submission Queue host software
issues a Delete I/O Submission Queue command for that queue (refer to section 3.3.1.3).
Host software writes the Submission Queue Tail Doorbell and the Completion Queue Head Doorbell (refer
to the Transport Specific Controller Properties  section in the NVMe over PCIe Transport Specification) to
communicate new values of the corresponding entry pointers to the controller. If host software writes an
invalid value to the Submission Queue Tail Doorbell or Completion Queue Head Doorbell property and an
Asynchronous Event Request command is outstanding, then an asynchronous event is posted to the Admin
Completion Queue with a status code of Invalid Doorbell Write Value. The associated queue is then deleted
and recreated by host software. For a Submission Queue that experiences this error, the controller may
complete previously consumed commands; no additional commands are consumed. This condition may be
caused by host software attempting to add an entry to a full Submission Queue or remove an entry from an
empty Completion Queue.
Host software checks completion queue entry Phase Tag (P) bits in memory to determine whether new
completion queue entries have been posted (refer to section 4.2.4). The Completion Queue Tail pointer is
only used internally by the controller and is not visible to the host. The controller uses the SQ Head Pointer
(SQHD) field in completion queue entries to communicate new values of the Submission Queue Head
Pointer to the host. A new SQHD value indicates that submission queue entries have been consumed, but
does not indicate either execution or completion of any command. Refer to section 4.2.
A submission queue entry is submitted to the controller when the host writes the associated Submission
Queue Tail Doorbell with a new value that indicates that the Submission Queue Tail Pointer has moved to
or past the slot in which that submission queue entry was placed. A Submission Queue Tail Doorbell write
may indicate that one or more submission queue entries have been submitted.
A submission queue entry has been consumed by the controller when a completion queue entry is posted
that indicates that the Submission Queue Head Pointer has moved past the slot in which that submission
queue entry was placed. A completion queue entry may indicate that one or more submission queue entries
have been consumed.
A completion queue entry is posted to the Completion Queue when the controller write of that completion
queue entry to the next free Completion Queue slot inverts the Phase Tag (P) bit from its previous value in
memory (refer to section 4.2.4). The controller may generate an interrupt to the host to indicate that one or
more completion queue entries have been posted.
A completion queue entry has been consumed by the host when the host writes the associated Completion
Queue Head Doorbell with a new value that indicates that the Completion Queue Head Pointer has moved
past the slot in which that completion queue entry was placed. A Completion Queue Head Doorbell write
may indicate that one or more completion queue entries have been consumed.
Once a submission queue entry or a completion queue entry has been consumed, the slot in which it was
placed is free and available for reuse. Altering a submission queue entry after that entry has been submitted
but before that entry has been consumed results in undefined behavior. Altering a completion queue entry
after that entry has been posted but before that entry has been consumed results in undefined behavior.
3.3.1.2.1 Completion Queue Flow Control
If there are no free slots in a Completion Queue, then the controller shall not post status to that Completion
Queue until slots become available. In this case, the controller may stop processing additional submission
queue entries associated with the affected Completion Queue until slots become available. The controller
shall continue processing for other Submission Queues not associated with the affected Completion Queue.
