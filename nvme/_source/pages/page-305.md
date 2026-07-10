---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 305
headings: []
---

# Source page 305

NVM Express® Base Specification, Revision 2.1

283
Figure 291: Sanitize Status Log Page
Bytes Description
27:24
Estimated Time For Block Erase With No -Deallocate Media Modification  (ETBENMM): This field
shall indicate the number of seconds required to complete sanitize processing (i.e., the time difference
between entering and exiting the Restricted Processing state or the Unrestricted Processing state) of a
Block Erase sanitize operation, inc luding associated additional media modification, in the background
(refer to section 5.1.22) when:
a) the No-Deallocate After Sanitize bit was set to ‘1’ in the Sanitize command that requested the
Block Erase sanitize operation; and
b) the No-Deallocate Modifies Media After Sanitize field (refer to Figure 312) is set to 10b.
A value of 0h indicates that the sanitize processing is expected to be completed in the background when
the Sanitize command that started that operation is completed. A value of FFFFFFFFh indicates that no
time period is reported.
31:28
Estimated Time For Crypto Erase With No -Deallocate Media Modification (ETCENMM): This field
shall indicate the number of seconds required to complete sanitize processing (i.e., the time difference
between entering and exiting the Restricted Processing state or the Unrestricted Processing state) of a
Crypto Erase sanitize operation, including associated additional media modification, in the background
(refer to section 5.1.22) when:
a) the No-Deallocate After Sanitize bit was set to ‘1’ in the Sanitize command that requested the
Crypto Erase sanitize operation; and
b) the No-Deallocate Modifies Media After Sanitize field (refer to Figure 312) is set to 10b.
A value of 0h indicates that the sanitize processing is expected to be completed in the background when
the Sanitize command that started that operation is completed. A value of FFFFFFFFh indicates that no
time period is reported.
35:32
Estimated Time For Post -Verification Deallocation State  (ETPVDS): This field shall indicate the
number of seconds required to deallocate all media allocated for user data after exiting the Media
Verification state (i.e., the time difference between entering and exiting the Post -Verification
Deallocation state), if that state is entered as part of the sanitize operation. A value of FFFFFFFFh
indicates that no time period is reported.
36
Sanitize State Information (SSI):  This field indicates additional information about the state of
sanitization.
Bits Description
7:4
Failure State (FAILS): If the SOS field is set to 011b (i.e., Sanitize Failed), then this field
shall indicate the state of the Sanitize Operation State Machine (refer to Figure 648) in
which the failure occurred.
The values of this field are sanitize states, defined in the description of the SANS field.
If the VERS bit is cleared to ‘0’, then this field shall be cleared to 0h.
If the SOS field is set to a value other than 011b (i.e., Sanitize Failed), then this field shall
be cleared to 0h.
3:0
Sanitize State (SANS): This field shall indicate the current state of the Sanitize Operation
State Machine (refer to Figure 648).
If the VERS bit is cleared to ‘0’, then this field shall be cleared to 0h.
Value State Reference
0h Idle 8.1.24.3.1
1h Restricted Processing 8.1.24.3.2
2h Restricted Failure 8.1.24.3.3
3h Unrestricted Processing 8.1.24.3.4
4h Unrestricted Failure 8.1.24.3.5
5h Media Verification 8.1.24.3.6
6h Post-Verification Deallocation 8.1.24.3.7
All other values Reserved


511:37 Reserved
