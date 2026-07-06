---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 337
headings: []
---

# Source page 337

NVM Express® Base Specification, Revision 2.1

315
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
03
Verification Support (VERS): If this bit is set to ‘1’, then the controller
supports the Media Verification state and the Post -Verification
Deallocation state. If this bit is cleared to ‘0’, then the controller does not
support the Media Verification state and does not support the Pos t-
Verification Deallocation state.
If the BES bit is cleared to ‘0’ and the CES bit is cleared to ‘0’, then this bit
shall be cleared to ‘0’.
02
Overwrite Support (OWS):  If this bit is  set to ‘1’, then the controller
supports the Overwrite sanitize operation. If this bit is cleared to ‘0’, then
the controller does not support the Overwrite sanitize operation.
01
Block Erase Support (BES):  If this bit is  set to ‘1’, then the controller
supports the Block Erase sanitize operation. If this bit is cleared to ‘0’, then
the controller does not support the Block Erase sanitize operation.
00
Crypto Erase Support (CES):  If this bit is  set to ‘1’, then the controller
supports the Crypto Erase sanitize operation. If this bit is  cleared to ‘0’,
then the controller does not support the Crypto Erase sanitize operation.

335:332 O O R
Host Memory Buffer Minimum Descriptor Entry Size (HMMINDS): This field indicates
the minimum usable size of a Host Memory Buffer Descriptor Entry in 4  KiB units. If this
field is cleared to 0h, then the controller does not indicate any limitations on the Host
Memory Buffer Descriptor Entry size.
337:336 O O R
Host Memory Maximum Descriptors Entries (HMMAXD ): This field indicates the
number of usable Host Memory Buffer Descriptor Entries. If this field is cleared to 0h,
then the controller does not indicate a maximum number of Host Memory Buffer
Descriptor Entries.
339:338 O O R
NVM Set Identifier Maximum (NSETIDMAX):  This field defines the maximum value of
a valid NVM Set Identifier for any controller in the NVM subsystem. The number of NVM
Sets supported by the NVM subsystem is less than or equal to NSETIDMAX.
341:340 O O R
Endurance Group Identifier Maximum (ENDGIDMAX): This field defines the maximum
value of a valid Endurance Group Identifier for any controller in the NVM subsystem. The
number of Endurance Groups supported by the NVM subsystem is less than or equal to
ENDGIDMAX.
342 O O R
ANA Transition Time (ANATT): This field indicates the maximum amount of time, in
seconds, for a transition between ANA states or the maximum amount of time, in
seconds, that the controller reports the ANA change state. If the controller supports
Asymmetric Namespace Access Reporting (refer to the CMIC field in Figure 312), then
this field shall be set to a non -zero value. If the controller does not support Asymmetric
Namespace Access Reporting, then this field shall be cleared to 0h. Refer to section
8.1.2.4.
