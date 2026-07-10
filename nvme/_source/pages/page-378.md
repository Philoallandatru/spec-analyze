---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 378
headings: []
---

# Source page 378

NVM Express® Base Specification, Revision 2.1

356
o the Sequence Indicator (SEQIND) field (refer to Figure 353) set to 01b; and
o the Controller Identifier field set to the controller whose state is being set,
 to specify to the controller that the command is the first of a sequence of Migration Send commands
of controller state data being transferred.
• The host may submit Migration Send commands with:
o the Select field set to Set Controller State (i.e., 2h);
o the SEQIND field cleared to 00b; and
o the Controller Identifier field set to the controller whose state is being set,
to specify to the controller that the command is not the last of the sequence of Migration Send
commands of controller state data being transferred, if any.
• The host is required to wait for the completion queue entries to be posted for the previous submitted
Migration Send commands in the sequence of Migration Send commands.
• Finally, the host submits the last Migration Send command with:
o the Select field set to Set Controller State (i.e., 2h);
o the SEQIND field set to 10b; and
o the Controller Identifier field set to the controller whose state is being set,
to specify to the controller that this is the last command of the sequence of Migration Send commands of
the controller state data being transferred. This last Migration Send command may or may not have the
Number of Dwords (NUMD) field cleared to 0h.
If a sequence of Migration Send commands of the controller state data being transferred and the controller
processes a Migration Send command with:
• the Select field set to Set Controller State (i.e., 2h);
• the Sequence Indicator (SEQIND) field (refer to Figure 353) set to 01b; and
• the Controller Identifier field set to the controller whose state is being set,
then, the command is specifying a new sequence of Migration Send commands of the controller state data
is being transferred and the previous sequence of Migration Send commands of the controller state data is
discarded.
If a sequence of Migration Send commands of the controller state data is not being transferred to the
specified controller and the controller processes a Migration Send command with the Sequence Indicator
(SEQIND) field (refer t o Figure 353) not set to 01b, clearer to 0b; the controller shall abort that command
with a status code of Command Sequence Error.
If a single Migration Send command is submitted to transfer the entire controller state, then the host submits
a Migration Send command with the SEQIND field set to 11b.
