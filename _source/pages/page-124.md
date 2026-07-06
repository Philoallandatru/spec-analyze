---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 124
headings: ["3.5 Controller Initialization", "3.5.1 Memory-based Controller Initialization (PCIe)"]
---

# Source page 124

NVM Express® Base Specification, Revision 2.1

102
• receives a successful completion with Immediate Abort Not Performed bit cleared to ‘0’ in Dword 0
of the completion queue entry for an Abort command specifying that outstanding command (refer
to section 5.1.1);
• receives a successful completion with the Commands Aborted field set to 1h for a Cancel command
with an Action Code of Single Command Cancel and specifying that outstanding command (refer
to section 7.1);
• reads a CSTS.RDY bit value that indicates a controller is not able to process commands except for
Fabrics commands (i.e., a value of ‘0’), if that outstanding command is not a Fabrics command
(refer to section 3.7.2);
• reads a CSTS.SHST field value that indicates that the controller shutdown is complete (i.e., a value
of 10b), if that outstanding command is not a Fabrics command (refer to section 3.6.1 for memory-
based transports and section 3.6.2 for message-based transports);
• if using a memory-based transport, receives a successful completion for a Delete I/O Submission
Queue command if that outstanding command was sent on the deleted I/O Submission queue
(refer to section 3.3.1.3);
• if using a message-based transport:
o receives a successful completion for a Disconnect command if that outstanding command was
sent on the same I/O queue as the Disconnect command (refer to section 3.3.2.4); or
o restores communication to the same controller after losing communication to that controller
(refer to section 3.9.5).
If an outstanding command ceases to be outstanding for one of these reasons, then further controller
processing of that command is no longer possible.
3.5 Controller Initialization
This section describes the recommended procedure for initializing a controller.
 Memory-based Controller Initialization (PCIe)
Upon completion of  the transport-specific controller initialization steps defined within the relevant NVMe
Transport binding specification, the host should perform the following sequence of actions to initialize the
controller to begin executing commands:
1. The host waits for the controller to indicate that any previous reset is complete by waiting for
CSTS.RDY to become ‘0’;
2. The host configures t he Admin Queue by setting the Admin Queue Attributes (AQA), Admin
Submission Queue Base Address (ASQ), and Admin Completion Queue Base Address (ACQ) to
appropriate values;
3. The host determines the supported I/O Command Sets  by checking the state of CAP.CSS and
appropriately initializing CC.CSS as follows:
a. If the CAP.CSS.NOIOCSS bit is set to ‘1’, then the CC.CSS field should be set to 111b;
b. If the CAP.CSS.IOCSS bit is set to ‘1’, then the CC.CSS field should be set to 110b; and
c. If the CAP.CSS.IOCSS bit is cleared to ‘0’ and  the CAP.CSS.NCSS bit is set to ‘1’, then the
CC.CSS field should be set to 000b;
4. The controller settings should be configured. Specifically:
a. The arbitration mechanism should be selected in CC.AMS; and
b. The memory page size should be initialized in CC.MPS;
5. The host enables the controller by setting CC.EN to ‘1’;
6. The host waits for the controller to indicate that the controller is ready to process commands. The
controller is ready to process commands when CSTS.RDY is set to ‘1’;
7. The host determines the configuration of the controller by issuing the Identify command specifying
the Identify Controller data structure (i.e., CNS 01h);
