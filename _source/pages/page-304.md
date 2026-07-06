---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 304
headings: []
---

# Source page 304

NVM Express® Base Specification, Revision 2.1

282
Figure 291: Sanitize Status Log Page
Bytes Description
a) the most recent sanitize operation for which No -
Deallocate After Sanitize (refer to section 5.1.22) was
requested has completed successfully with deallocation of
all media allocated for user data (refer to section
5.1.25.1.15); and
b) the Sanitize Operation State Machine is in the Idle state,
then this value shall be reported. Otherwise, this value shall not be
reported.
101b to 111b Reserved


07:04
Sanitize Command Dword 10 Information (SCDW10): This field shall indicate  the value of the
Command Dword 10 field of the Sanitize command that started the sanitize operation whose status is
reported in the SSTAT field. Refer to Figure 372.
11:08
Estimated Time For Overwrite  (ETO): This field shall indicate  the number of seconds required to
complete sanitize processing (i.e., the time difference between entering and exiting the Restricted
Processing state or the Unrestricted Processing state) of an Overwrite sanitize operation with 16 passes
in the background (refer to  section 5.1.22) when the No-Deallocate Modifies Media After Sanitize field
(refer to Figure 312) is set to a value other than 10b.
A value of 0h indicates that the sanitize processing is expected to be completed in the background when
the Sanitize command that started that operation is completed. A value of FFFFFFFFh indicates that no
time period is reported.
15:12
Estimated Time For Block Erase (ETBE): This field shall indicate the number of seconds required to
complete sanitize processing (i.e., the time difference between entering and exiting the Restricted
Processing state or the Unrestricted Processing state) of a  Block Erase sanitize operation in the
background (refer to section 5.1.22) when the No-Deallocate Modifies Media After Sanitize field (refer
to Figure 312) is set to a value other than 10b.
A value of 0h indicates that the sanitize processing is expected to be completed in the background when
the Sanitize command that started that operation is completed. A value of FFFFFFFFh indicates that no
time period is reported.
19:16
Estimated Time For Crypto Erase (ETCE): This field shall indicate the number of seconds required to
complete sanitize processing (i.e., the time difference between entering and exiting the Restricted
Processing state or the Unrestricted Processing state) of a Crypto Erase sanitize operation in t he
background (refer to section 5.1.22) when the No-Deallocate Modifies Media After Sanitize field (refer
to Figure 312) is set to a value other than 10b.
A value of 0h indicates that the sanitize processing is expected to be completed in the background when
the Sanitize command that started that operation is completed. A value of FFFFFFFFh indicates that no
time period is reported.
23:20
Estimated Time For Overwrite With No-Deallocate Media Modification (ETODMM): This field shall
indicate the number of seconds required to complete sanitize processing (i.e., the time difference
between entering and exiting the Restricted Processing state or the Unrestricted Processing state) of
an Overwrite sanitize operation, including associated additional media modification, in the background
(refer to section 5.1.22) when:
a) the No-Deallocate After Sanitize bit was set to ‘1’ in the Sanitize command that requested the
Overwrite sanitize operation; and
b) the No-Deallocate Modifies Media After Sanitize field (refer to Figure 312) is set to 10b.
A value of 0h indicates that the sanitize processing is expected to be completed in the background when
the Sanitize command that started that operation is completed. A value of FFFFFFFFh indicates that no
time period is reported.
