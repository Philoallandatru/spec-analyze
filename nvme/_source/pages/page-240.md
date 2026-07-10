---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 240
headings: []
---

# Source page 240

NVM Express® Base Specification, Revision 2.1

218
Figure 212: Self-test Result Data Structure
Bytes Description
00
Device Self-test Status (DSTS): This field indicates the device self-test code and the status of
that operation.
Bits Description
7:4
Self-test Code (DSTC): This field indicates the Self -test Code value that was
specified in the Device Self -test command that started the device self -test operation
that this Self-test Result Data Structure describes.
Value Definition
0h Entry is empty
1h Short device self-test operation
2h Extended device self-test operation
3h Host-Initiated Refresh operation
4h to Dh Reserved
Eh Vendor specific
Fh Reserved

3:0
Device Self-test Result (DSTR): This field indicates the result of the device self-test
operation that this Self-test Result Data Structure describes.
Value Definition
0h Operation completed without error
1h Operation was aborted by a Device Self-test command
2h Operation was aborted by a Controller Level Reset
3h Operation was aborted due to a removal of a namespace from the
namespace inventory
4h Operation was aborted due to the processing of a Format NVM
command
5h
A fatal error or unknown test error occurred while the controller
was executing the device self-test operation and the operation did
not complete
6h Operation completed with a segment that failed and the segment
that failed is not known
7h Operation completed with one or more failed segments and the
first segment that failed is indicated in the Segment Number field
8h Operation was aborted for unknown reason
9h Operation was aborted due to a sanitize operation
Ah to Eh Reserved
Fh Entry is empty (i.e., does not contain a test result)


01
Segment Number  (SEGN): This field indicates the segment number (refer to section 8.1.7)
where the first self-test failure occurred. If Device Self-test Status field bits [3:0] are not set to 7h,
then this field should be ignored by the host.
02
Valid Diagnostic Information (VDINFO): This field indicates the diagnostic failure information
that is reported.
Bits Description
7:4 Reserved
3 SC Valid (SCVLD): If this bit is set to ‘1’, then the contents of Status Code field are
valid. If this bit is cleared to ‘0’, then the contents of the Status Code field are invalid.
2
SCT Valid (SCTVLD): If this bit is set to ‘1’, then the contents of the Status Code Type
field are valid. If this bit is  cleared to ‘0’, then the contents of the Status Code Type
field are invalid.
1 FLBA Valid (FVLD): If this bit is set to ‘1’, then the contents of the Failing LBA field
are valid. If this bit is cleared to ‘0’, then the contents of the Failing LBA field are invalid.
0
NSID Valid (NSIDVLD): If this bit is  set to ‘1’, then the contents of the Namespace
Identifier field are valid. If this bit is cleared to ‘0’, then the contents of the Namespace
Identifier field are invalid.

03 Reserved
