---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 211
headings: []
---

# Source page 211

NVM Express® Base Specification, Revision 2.1

189
Figure 171: Device Self-test – Command Dword 10
Bits Description
03:00
Self-test Code (STC): This field specifies the action taken by the Device Self-test command.
Value Definition
0h Reserved
1h Start a short device self-test operation
2h Start an extended device self-test operation
3h Start a Host-Initiated Refresh operation
4h to Dh Reserved
Eh Vendor specific . Additional information may be provided in Command Dword 15
(refer to Figure 172).
Fh Abort device self-test operation


Figure 172: Device Self-test – Command Dword 15
Bits Description
31:00 Device Self-test Parameter (DSTP): If the Self-test Code field is set to Eh (refer to Figure 171), then
this field is vendor specific; otherwise this field is reserved.
The processing of a Device Self-test command and interactions with a device self-test operation already in
progress is defined in Figure 173.
Figure 173: Device Self-test – Command Processing
Self-test in
Progress 1
Self-test Code value in new
Drive Self-test command Controller Action
Yes
1h – Short device self-test Abort the new Device Self -test command with a status
code of Device Self-test in Progress. 2h – Extended device self-test
3h – Host-Initiated Refresh
Eh – Vendor specific Vendor specific
Fh – Abort device self-test
The controller takes the following actions in order:
1. Abort device self-test operation in progress;
2. Create log entry in the Newest Self -test Result
Data Structure in the Device Self-test log page;
3. Set the Device Selt-test Result field of the current
Device Self -test Status field in the Device Self -
test log page to 0h; and
4. Completes command successfully.
No 1h – Short device self-test
The controller takes the following actions in order:
1. Validate the command parameters;
2. Set the Device Selt-test Result field of the current
Device Self -test Status field in the Device Self -
test log page to 1h;
3. Start a device self-test operation; and
4. Completes command successfully.
