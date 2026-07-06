---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 336
headings: []
---

# Source page 336

NVM Express® Base Specification, Revision 2.1

314
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
327:326 O O R
Maximum Thermal Management Temperature (MXTMT):  This field indicates the
maximum temperature, in Kelvins, that the host may request in the Thermal Management
Temperature 1 field and Thermal Management Temperature 2 field of the Set Features
command with the Feature Identifier set to 10h. A value of 0h indicates that the controller
does not report this field or the host controlled thermal management feature is not
supported.
331:328 O O R
Sanitize Capabilities (SANICAP): This field indicates attributes for sanitize operations.
If the Sanitize command is supported, then this field shall be non -zero. If the Sanitize
command is not supported, then this field shall be cleared to 0h. Refer to section 8.1.24.
Bits Description
31:30
No-Deallocate Modifies Media After Sanitize (NODMMAS): This field
indicates if media is additionally modified by the controller as part of
sanitize processing that had been started by a Sanitize command with the
No-Deallocate After Sanitize bit set to ‘1’.
Value Definition
00b
Additional media modification as part of sanitize
processing is not defined. Only controllers compliant with
NVM Express Base Specification, Revision 1.3 and
earlier, or controllers that do not support the Sanitize
command are allowed to return this value.
01b Media shall not be additionally modified by the controller
as part of sanitize processing.
10b
Media shall be additionally modified by the controller as
part of sanitize processing that had been started by a
Sanitize command with the No-Deallocate After Sanitize
bit set to ‘1’.
11b Reserved

29
No-Deallocate Inhibited (NDI): If this bit is set to ‘1’ and the No -
Deallocate Response Mode bit (refer to Figure 408) is set to ‘1’, then the
controller deallocates all media allocated for user data before the
Restricted Processing:Idle transition occurs or the Unrestricted
Processing:Idle transition occurs (refer to section 8.1.24.3), even if the No-
Deallocate After Sanitize bit is set to ‘1’ in a Sanitize command.
If:
a) this bit is set to ‘1’;
b) the No -Deallocate After Sanitize bit is set to ‘1’ in a Sanitize
command, and:
1) the No-Deallocate Response Mode bit is cleared to ‘0’; or
2) the Sanitize Config Feature (refer to section 5.1.25.1.15) is
not supported,
then the controller aborts the Sanitize command with a status code of
Invalid Field in Command.
If the No -Deallocate After Sanitize bit is cleared to ‘0’ in a Sanitize
command, then the value of this bit has no effect on the processing of that
Sanitize command or on the sanitize operation that is started by that
Sanitize command.
If this bit is cleared to ‘0’, then the controller supports the No -Deallocate
After Sanitize bit in a Sanitize command.
28:04 Reserved
