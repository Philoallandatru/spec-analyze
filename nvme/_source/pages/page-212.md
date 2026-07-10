---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 212
headings: ["5.1.5.1 Command Completion", "5.1.6 Directive Receive command"]
---

# Source page 212

NVM Express® Base Specification, Revision 2.1

190
Figure 173: Device Self-test – Command Processing
Self-test in
Progress 1
Self-test Code value in new
Drive Self-test command Controller Action
2h – Extended device self-test
The controller takes the following actions in order:
1. Validate the command parameters;
2. Set the Device Selt-test Result field of the current
Device Self -test Status field in the Device Self -
test log page to 2h;
3. Start a device self-test operation; and
4. Completes command successfully.
3h – Host-Initiated Refresh
The controller takes the following actions in order:
1. Validate the command parameters;
2. Set the Device Selt-test Result field of the current
Device Self -test Status field in the Device Self -
test log page to 3h;
3. Start a Host-Initiated Refresh operation; and
Complete the command successfully.
Eh – Vendor specific Vendor specific
Fh – Abort device self-test Completes command successfully. The Device Self -test
log page is not modified.
Notes:
1. If the Single Device Self -test Operation (SDSO) bit is cleared to ‘0’ in the Device Self -test Options (DSTO) of
the Identify Controller data structure (refer to Figure 312), then the Self-test in Progress column represents that
a device self-test operation is in progress on the controller that the new Device Self-test command was received
on. If the SDSO bit is set to ‘1’, then the Self-test in Progress column represents that a device self-test operation
is in progress on the NVM subsystem.
 Command Completion
A completion queue entry is posted to the Admin Completion Queue after the appropriate actions are taken
as specified in Figure 173. Device Self-test command specific status values are defined in Figure 174.
Figure 174: Device Self-test – Command Specific Status Values
Value Description
1Dh Device Self-test in Progress:  The controller or NVM subsystem already has a device self -test
operation in process.
 Directive Receive command
The Directive Receive command returns a data buffer that is dependent on the Directive Type. Refer to
section 8.1.8.
The Directive Receive command uses the Data Pointer, Command Dword 10, and Command Dword 11
fields. Command Dword 12 and Dword 13 may be used based on the Directive Type field and the Directive
Operation field. All other command specific fields are reserved.
If the Number of Dwords (NUMD) field corresponds to a length that is less than the size of the data structure
to be returned, then only that specified portion of the data structure is transferred. If the NUMD field
corresponds to a length that is greater than the size of the associated data structure, then the entire contents
of the data structure are transferred and no additional data is transferred.
Figure 175: Directive Receive – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.
