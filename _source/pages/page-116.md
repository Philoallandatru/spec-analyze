---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 116
headings: []
---

# Source page 116

NVM Express® Base Specification, Revision 2.1

94
Queue by setting the Disconnect Command Support (DCS) bit to ‘1’ in the OFCS field of the Identify
Controller data structure (refer to Figure 312).
If both the host and the controller support deletion of an individual I/O Queue, then the termination of an
individual I/O Queue impacts only that I/O Queue (i.e., the association and all other I/O Queues and their
associated NVMe Transport connections are  not impacted). If either the host or the controller does not
support deletion of an individual I/O Queue, then the deletion of an individual I/O Queue or the termination
of an NVMe Transport connection causes the association to be terminated.
NVMe over Fabrics uses the Disconnect command to delete an Individual I/O Queue. This command is
sent on the I/O Submission Queue to be deleted and affects only that I/O Submission Queue and its
associated I/O Completion Queue (i.e., other I/O Queues are n ot affected). To delete an I/O Queue, the
NVMe Transport connection for that I/O Queue is used. If all Queues associated with an NVMe Transport
connection are deleted, then the NVMe Transport connection may be deleted after completion of the
Disconnect command. Actions necessary to delete the NVMe Transport connection are transport specific.
The association between the host and the controller is not affected.
If a Disconnect command returns a status code other than success, the host may delete an I/O Queue
using other methods including:
• waiting a vendor specific amount of time and retry the Disconnect command;
• deleting the NVMe Transport connection (note: this may impact other I/O Queues);
• performing a Controller Level Reset (note: this impacts other I/O Queues); or
• ending the host to controller association.
If the transport requires a separate NVMe Transport connection for each Admin and I/O Queue (refer to
section 3.3.2.2), then the host should not delete an NVMe Transport connection until after:
• a Disconnect command has been submitted to the I/O Submission Queue; and
• the response for that Disconnect command has been received by the host on the corresponding
I/O Completion Queue or a vendor specific timeout (refer to section 3.9) has occurred while waiting
for that response.
If the transport requires a separate NVMe Transport connection for each Admin and I/O Queue, then the
controller should not delete an NVMe Transport connection until after:
• a Disconnect command has been received on the I/O Submission Queue and processed by the
controller;
• the responses for commands received by the controller on that I/O Submission Queue prior to
receiving the Disconnect command have been sent to the host on the corresponding I/O
Completion Queue; and
• the resulting response for that Disconnect command has been sent to the host on the
corresponding I/O Completion queue (i.e., this response is the last response sent). It is
recommended that the controller delay destroying the NVMe Transport connection to allow time for
the Disconnect command response to be received by the host (e.g., a transport specific event
occurs or a transport specific time period elapses).
If the transport utilizes the same NVMe Transport connection for all Admin and I/O Queues associated with
a particular controller (refer to section 3.3.2.2), then the deletion of an individual I/O Queue has no impact
on the NVMe Transport connection.
A Disconnect command is the last I/O Submission Queue entry processed by the controller for an I/O
Queue. Controller processing of the Disconnect command completes or aborts all commands on the I/O
Queue on which the Disconnect command was received. The co ntroller determines whether to complete
or abort each of those commands.  Until the controller sends a successful completion for a Disconnect
command, outstanding commands may continue being processed by the controller. The controller ensures
that there is no further processing of any command sent on that I/O Queue after posting the completion
queue entry for the Disconnect command as described in section 6.4.
