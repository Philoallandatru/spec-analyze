---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 301
headings: ["5.1.12.1.31.1.3 Controller Level Reset Modified Reclaim Unit Handles (Event Type 2h)", "5.1.12.1.31.1.4 Invalid Placement Identifier (Event Type 3h)", "5.1.12.1.31.2 Controller Events", "5.1.12.1.31.2.1 Media Reallocated (Event Type 80h)", "5.1.12.1.31.2.2 Implicitly Modified Reclaim Unit Handle (Event Type 81h)", "5.1.12.1.32 Reservation Notification (Log Page Identifier 80h)"]
---

# Source page 301

NVM Express® Base Specification, Revision 2.1

279
This FDP event indicates that the capacity of a Reclaim Unit was not completely written within the time
period defined in the Estimated Reclaim Unit Time Limit field (refer to Figure 280) due to the controller
modifying the Reclaim Unit Handle to reference a different Reclaim Unit.
5.1.12.1.31.1.3 Controller Level Reset Modified Reclaim Unit Handles (Event Type 2h)
This FDP event indicates that one or more Reclaim Unit Handles were modified to reference a different
Reclaim Unit due to a Controller Level Reset.
In the FDP event, the NSID field is reserved and the Placement Identifier field is reserved.
5.1.12.1.31.1.4 Invalid Placement Identifier (Event Type 3h)
This FDP event indicates that a write command specified an invalid Placement Identifier (refer to Figure
282 and Figure 283) due to an invalid Placement Handle or an invalid Reclaim Group Identifier.
 Controller Events
5.1.12.1.31.2.1 Media Reallocated (Event Type 80h)
This FDP event indicates that user data contained in a Reclaim Unit was written by the host specifying a
Reclaim Unit Handle with a Reclaim Unit Handle type of Initially Isolated and was moved by the controller
to a different Reclaim Unit (e.g., controller performed garbage collection).
The Event Type Specific fiel d is I/O Command Set specific. Refer to the applicable I/O Command Set
specification to determine if this log page is supported and the definition of the Event Type Specific field for
this FDP event.
5.1.12.1.31.2.2 Implicitly Modified Reclaim Unit Handle (Event Type 81h)
This FDP event indicates that the controller modified a Reclaim Unit Handle to reference a different Reclaim
Unit where that modification was not due to any host action (e.g., a write command that has a size greater
than the remaining capacity of the initial Reclaim Unit being written).
5.1.12.1.32 Reservation Notification (Log Page Identifier 80h)
The Reservation Notification log page reports one log page from a time ordered queue of Reservation
Notification log pages, if available. A new Reservation Notification log page is created and added to the end
of the queue of reservation notifications whenever an unmasked reservation notification occurs on any
namespace that is attached to the controller. The Get Log Page command:
• returns a data buffer containing a log page corresponding to the oldest log page in the reservation
notification queue (i.e., the log page containing the lowest Log Page Count field; accounting for
wrapping); and
• removes that Reservation Notification log page from the queue.
If there are no available Reservation Notification log page entries when a Get Log Page command is issued,
then an empty log page (i.e., all fields in the log page cleared to 0h) shall be returned.
If the controller is unable to store a reservation notification in the Reservation Notification log page due to
the size of the queue, that reservation notification is lost. If a reservation notification is lost, then the
controller shall increment the Log Page Count field of the last reservation notification in the queue (i.e., the
Log Page Count field in the last reservation notification in the queue shall contain the value associated with
the most recent reservation notification that has been lost).
The format of the log page is defined in Figure 290.
