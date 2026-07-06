---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 241
headings: ["5.1.12.1.8 Telemetry Host-Initiated (Log Page Identifier 07h)"]
---

# Source page 241

NVM Express® Base Specification, Revision 2.1

219
Figure 212: Self-test Result Data Structure
Bytes Description
11:04
Power On Hours (POH):  This field indicates the number of power -on hours at the time the
device self-test operation was completed or aborted. This does not include time that the controller
was powered and in a low power state condition.
15:12 Namespace Identifier (NSID): This field indicates the namespace that the Failing LBA occurred
on. The contents of this field are valid only when the NSID Valid bit is set to ‘1’.
23:16
Failing LBA (FLBA): This field is I/O Command Set specific and is described in the applicable
I/O Command Set specification.
NOTE: The original field name has been retained for historical continuity.
24
Status Code Type  (STCT): This field may contain additional information related to errors or
conditions.
Bits Description
7:3 Reserved
2:0
Additional Status Code Info (ASTCI): This field may contain additional
information relating to errors or conditions that occurred during the device self -
test operation represented in the same format used in the Status Code Type field
of the completion queue entry (refer to Figure 101). The contents of this field are
valid only when the SCT Valid bit is set to ‘1’.

25
Status Code (STC): This field may contain additional information relating to errors or conditions
that occurred during the device self -test operation represented in the same format used in the
Status Code field of the completion queue entry (refer to section 4.2.3). The contents of this field
are valid only when the SC Valid bit is set to ‘1’.
27:26 Vendor Specific (VS): This field is vendor specific.
5.1.12.1.8 Telemetry Host-Initiated (Log Page Identifier 07h)
This log page consists of a header  (i.e., bytes 511:0 of the log page) describing the log and zero or more
Telemetry Data Blocks (refer to section  8.1.27). The header shall always be available even if there is no
Telemetry Host-Initiated Data available. All Telemetry Data Blocks are 512 bytes in size. The controller
shall initiate a capture of the internal controller state or internal NVM subsystem state to this log page if the
controller processes a Get Log Page command for this log with the Create Telemetry Host -Initiated Data
bit set to ‘1’ in the Log Specific Parameter field. If the host specifies a Log Page Offset Lower value that is
not a multiple of 512 bytes in the Get Log Page command for this log, then the controller shall abort the
command with a status code of Invalid Field in Command.
The Log Specific Parameter field in Command Dword 10 (refer to Figure 197) for this log page is defined
in Figure 213.
Figure 213: Telemetry Host-Initiated Log Specific Parameter Field
Bits Description
14:12 Reserved
11:09
Maximum Created Data Area (MCDA): If the MCDAS bit is set to ‘1’ in the LID Specific Parameter field
(refer to Figure 215) and the Create Telemetry Host-Initiated Data bit is set to ‘1’, then this field specifies
the data areas the host is requesting to be created in the log page. Data areas not requested shall not
be created.
Values Description
000b The controller determines the data areas to be created in the log page.
001b Data Area 1
010b Data Area 1 through Data Area 2
011b Data Area 1 through Data Area 3
100b Data Area 1 through Data Area 4
101b to 111b Reserved
If the MCDAS bit is cleared to ‘0’ or the Create Telemetry Host -Initiated Data bit is cleared to ‘0’, then
this field shall be ignored by the controller.
