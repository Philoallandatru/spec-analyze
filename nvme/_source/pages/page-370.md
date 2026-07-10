---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 370
headings: ["5.1.13.5 Command Completion", "5.1.14 Keep Alive command", "5.1.14.1 Command Completion", "5.1.15 Lockdown command"]
---

# Source page 370

NVM Express® Base Specification, Revision 2.1

348
Figure 336: Underlying Fabrics Transport Entry Data Structure
Bytes Description
513:512 Port ID of the Underlying Port (PIDUP): Refer to the PORTID field in Figure 294.
514 Transport Type (TRTYPE): Specifies the NVMe Transport Type. Refer to Figure 294.
515 Transport Address Family (ADRFAM): Specifies the Address Family. Refer Figure 294.
516 Transport Requirements (TREQ): This field indicates  requirements for the NVMe Transport. Refer to
Figure 294.
575:517 Reserved
 Command Completion
Upon completion of the Identify command, the controller posts a completion queue entry to the Admin
Completion Queue.
 Keep Alive command
The Keep Alive command  and the Keep Alive Timer feature (refer to section 5.1.25.1.8) are used by the
Keep Alive capability (refer to section 3.9). The controller should process the Keep Alive command as soon
as the command is fetched. If the Keep Alive Timer is active (refer to section 3.9.2), then upon processing
a Keep Alive command, the controller restarts the Keep Alive Timer.
All command specific fields are reserved.
 Command Completion
Upon completion of the Keep Alive command, the controller shall post a completion queue entry to the
Admin Completion Queue indicating the status for the command.
 Lockdown command
The Lockdown command is used to control the Command and Feature Lockdown capability (refer to section
8.1.5) which configures the prohibition or allowance of execution of the specified command or Set Features
command targeting a specific Feature Identifier.
After a successful completion of a Lockdown command prohibiting a command or Feature Identifier, all
controllers, if applicable, and all management endpoints, if applicable, in the NVM subsystem behave as
described in section 8.1.5.
The Lockdown command uses Command Dword 10 (refer to  Figure 337) and Command Dword 14 (refer
to Figure 338). All other command specific fields are reserved.
Figure 337: Lockdown – Command Dword 10
Bits Description
31:16 Reserved
15:08 Opcode or Feature Identifier (OFI): This field specifies the command opcode or Set Features Feature
Identifier identified by the Scope field.
07 Reserved
06:05
Interface (IFC): This field identifies the interfaces affected by this command. The actions of this command
apply if a command is received on the specified interfaces.
Value Affected Interfaces
00b Admin Submission Queue
01b Admin Submission Queue and out-of-band on a Management Endpoint
10b Out-of-band on a Management Endpoint
11b Reserved
