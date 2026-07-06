---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 517
headings: ["8.1.6 Controller Data Queue", "8.1.6.1 Controller Data Queue Usage"]
---

# Source page 517

NVM Express® Base Specification, Revision 2.1

495
 Controller Data Queue
A Controller Data Queue (CDQ) is used to post information from the controller to the host that is specific to
the type of queue being created (refer to Figure 165) by the Controller Data Queue command that specifies
the Create Controller Data Queue management operation (refer to section 5.1.4.1.1). A CDQ is a circular
buffer with a fixed slot size with entries posted by a controller.
A CDQ is deleted:
• by a Controller Data Queue command that specifies the Delete Controller Data Queue
management operation (refer to section 5.1.4.1.2);
• on a Controller Level Reset on the controller that processed the Controller Data Queue command
that created the CDQ; or
• by a Migration Send command that specifies the Suspend management operation and the Delete
User Data Migration Queue (DUDMQ) bit is set to ‘1’ (refer to section 5.1.17.1.1).
On the successful completion of a Controller Data Queue command, a Controller Data Queue Identifier
(CDQID) is returned in the completion queue entry (refer to Figure 168). A CDQID is used to identify the
created Controller Data Queue on the controller that processed the command. The returned value for that
CDQID is any value except the CDQID values for any Controller Data Queue that exists on the controller
that processed  the command (i.e., a controller is permitted to return the value of a CDQID value of a
Controller Data Queue that was previously deleted). The scope of a CDQID is per controller.
The CDQ Head pointer is updated by the host by issuing a Set Features command specifying the Controller
Data Queue feature and the Controller Data Queue Identifier (refer to section 5.1.25.1.23) after processing
an entry indicating the last free CDQ slot. A Controller Data Queue Phase Tag (CDQP) bit is defined in the
CDQ entry (refer to the applicable I/O command set specification) to indicate whether an entry has been
newly posted without the host relying on the Controller Data Queue Tail Pointer event (refer to Figure 154).
The Controller Data Queue Phase Tag bit enables the host to determine whether entries are new or not.
 Controller Data Queue Usage
The controller uses the current Tail entry pointer to identify the next open CDQ slot. The controller
increments the Tail entry pointer after placing the new entry to the next open CDQ slot. If the Tail entry
pointer increment exceeds the CDQ size , then the Tail entry pointer shall roll to zero. The controller may
continue to place entries in free CDQ slots as long as the Full queue condition is not met (refer to section
3.3.1.5). The controller shall take CDQ wrap conditions into account.
The host uses the current Head entry pointer to identify the slot containing the next entry to be consumed.
The host increments the Head entry pointer after consuming the next entry from the CDQ. If the Head entry
pointer increment exceeds the CDQ size, th e Head entry pointer shall roll to zero. The host may continue
to consume entries from the CDQ as long as the Empty queue condition is not met (refer to section 3.3.1.4).
Note: The host shall take CDQ wrap conditions into account.
A host issues a Set Features command specifying the Controller Data Queue feature to communicate a
new value of the Head entry pointer to the controller. If the host specifies an invalid value to the Head entry
pointer, then that Set Features command is aborted. This condition may be caused by a host attempting to
remove an entry from an empty CDQ.
A host checks a posted entries Controller Data Queue Phase Tag (CDQP) bits in memory to determine
whether new CDQ entries have been posted (refer to section 8.1.6.2). The CDQ Tail pointer is only used
internally by the controller and is not visible to the host.
An entry is posted to the CDQ when the controller write of that entry to the next free CDQ slot inverts the
Controller Data Queue Phase Tag (CDQP) bit from its previous value in memory (refer to section 8.1.6.2).
The controller generates a Controller Data Queue Tail Pointer event (refer to Figure 154) to the host to
indicate that the CDQ slot specified by the host in the Controller Data Queue feature (refer to section
5.1.25.1.23) has been posted if in the Controller Data Queue feature associated with the CDQ:
• the Tail pointer value matches the value in the Tail Pointer Trigger (TPT) field; and
