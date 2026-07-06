---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 389
headings: []
---

# Source page 389

NVM Express® Base Specification, Revision 2.1

367
Support for Sanitize commands in a Controller Memory Buffer (i.e., submitted to an Admin Submission
Queue in a Controller Memory Buffer or specifying an Admin Completion Queue in a Controller Memory
Buffer) is implementation specific. If an implementation does not support Sanitize commands in a Controller
Memory Buffer and a controller’s Admin Submission Queue or Admin Completion Queue is in the Controller
Memory Buffer, then the controller shall abort all Sanitize commands with a status code of Command Not
Supported for Queue in CMB.
All sanitize operations (i.e., Block Erase, Crypto Erase, and Overwrite) are performed in the background
(i.e., Sanitize command completion does not indicate sanitize operation completion). If a sanitize operation
starts as a result of a Sanitize command, then the controller shall complete that Sanitize command with a
status code of Successful Completion. If the controller completes a Sanitize command with any status code
other than Successful Completion, then the controller:
• shall not start the sanitize operation for that command;
• shall not modify the Sanitize Status log page; and
• shall not alter any user data.
The Sanitize command uses Command Dword 10 and Command Dword 11. All other command specific
fields are reserved.
Figure 372: Sanitize – Command Dword 10
Bits Description
31:11 Reserved
10
Enter Media Verification State (EMVS): If this bit is set to ‘1’, then the Media Verification state shall be
entered if sanitize processing completes successfully (i.e., the Global Data Erased (GDE) bit is set to ‘1’
(refer to Figure 291)).
If this bit is cleared to ‘0’, then this bit shall have no effect.
If the SANACT field does not specify starting a sanitize operation (i.e., is set to any value other than
010b, 011b, or 100b), then this bit shall be ignored by the controller.
09
No-Deallocate After Sanitize (NDAS): If this bit is set to ‘1’ and the No-Deallocate Inhibited bit (refer to
Figure 312) is cleared to ‘0’, then the controller shall not deallocate any media allocated for user data as
a result of successfully completing the sanitize operation. If this bit is:
a) cleared to ‘0’; or
b) set to ‘1’ and the No-Deallocate Inhibited bit is set to ‘1’,
then the controller should deallocate all media allocated for user data as a result of successfully
completing the sanitize operation.
If the SANACT field does not specify starting a sanitize operation (i.e., is set to any value other than
010b, 011b, or 100b), then this bit shall be ignored by the controller.
08
Overwrite Invert Pattern Between Passes (OIPBP): If this bit is set to ‘1’, then the Overwrite Pattern
shall be inverted between passes. If this bit is cleared to ‘0’, then the overwrite pattern shall not be
inverted between passes. If the Sanitize Action field is set to a value other than 011b (i.e., Overwrite),
then this bit shall be ignored by the controller.
07:04
Overwrite Pass Count (OWPASS): This field specifies the number of overwrite passes (i.e., how many
times the media is to be overwritten) using the data from the Overwrite Pattern field of this command. A
value of 0h specifies 16 overwrite passes. If the Sanitize Action field is set to a value other than 011b
(i.e., Overwrite), then this field shall be ignored by the controller.
03
Allow Unrestricted Sanitize Exit (AUSE):  If this bit is set to ‘1’, then the sanitize processing is
performed in unrestricted completion mode (i.e., in the Unrestricted Processing state; refer to section
8.1.24.3.4). If this bit is cleared to ‘0’, then the sanitize processing is performed in restricted completion
mode (i.e., in the Restricted Processing state; refer to section 8.1.24.3.2).
If the SANACT field does not specify starting a sanitize operation (i.e., is set to any value other than
010b, 011b, or 100b), then this bit shall be ignored by the controller.
