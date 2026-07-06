---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 690
headings: ["9 Error Reporting and Recovery", "9.1 Command and Queue Error Handling", "9.2 Media and Data Error Handling", "9.3 Memory Error Handling", "9.4 Internal Controller Error Handling", "9.5 Controller Fatal Status Condition"]
---

# Source page 690

NVM Express® Base Specification, Revision 2.1

668
9 Error Reporting and Recovery
9.1 Command and Queue Error Handling
In the case of serious error conditions, like Completion Queue Invalid, the operation of the associated
Submission Queue or Completion Queue may be compromised.  In this case, host software should delete
the associated Completion Queue and/or Submission Queue. The delete of a Submission Queue aborts all
outstanding commands, and deletion of either queue type releases resources associated with th at queue.
Host s oftware should recreate the Completion Queue and/or Submission Queue  to then continue with
operation.
In the case of serious error conditions for Admin commands, the entire controller should be reset using a
Controller Level Reset. The entire controller should also be reset if a completion is not received for the
deletion of a Submission Queue or Completion Queue.
For most command errors, there  is not an issue with the Submission Queue  and/or Completion Queue
itself. Thus, host software and the controller should continue to process commands.  It is at the discretion
of host software whether to retry the failed command; the Retry bit in the completion queue entry indicates
whether a retry of the failed command may succeed.
9.2 Media and Data Error Handling
In the event that the requested operation could not be performed to the NVM media, the particular command
is completed with a media error indicating the type of failure using the appropriate status code.
If a read error occurs during the processing of a command , (e.g. , End-to-end Guard Check Error,
Unrecovered Read Error), the controller may either stop the DMA transfer into the memory or transfer the
erroneous data to the memory. The host shall ignore the data in the memory locations for commands that
complete with such error conditions.
If a write error occurs during the processing of a command, (e.g., an internal error, End-to-end Guard Check
Error, End -to-end Application Tag Check Error), the controller may either stop or complete the DMA
transfer.
Additional I/O Command Set specific error handling is described within applicable I/O Command Set
specifications.
9.3 Memory Error Handling
For PCI Express implementations, memory errors such as target abort, master abort, and parity may cause
the controller to stop processing the currently executing command. These are serious errors that cannot be
recovered from without host software intervention.
A master abort or target abort error occurs when host software has provided, to the controller, the address
of memory that does not exist . When this occurs, the controller aborts the command with a Data Transfer
Error status code.
9.4 Internal Controller Error Handling
If a controller level failure (e.g., a DRAM failure) occurs during the processing of a command, then the
controller should abort the command with a status code of Internal Error. Any data transfer associated with
the command may be incomplete or incorrect,  and therefore any transferred data should not be used for
any purpose other than error reporting or diagnosis. The host may choose to re -submit the command or
indicate an error to the higher-level software.
9.5 Controller Fatal Status Condition
If the controller has a serious error condition and is unable to communicate with host software via
completion queue entries  in the Admin Completion Queue or I/O Completion Queues, then the controller
may set the Controller Fatal Status (CSTS.CFS) bit to ‘1’ (refer to section 3.1.4.6). This indicates to host
