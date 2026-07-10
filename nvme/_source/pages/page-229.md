---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 229
headings: ["5.1.12.1.2 Error Information (Log Page Identifier 01h)"]
---

# Source page 229

NVM Express® Base Specification, Revision 2.1

207
5.1.12.1.2 Error Information (Log Page Identifier 01h)
This log page is used to describe extended error information for a command that completed with error  or
report an error that is not specific to a particular command . Extended error information is provided when
the More (M) bit is set to ‘1’ in the Status field for the completion queue entry associated with the command
that completed with error or as part of an asynchronous event with an Error status type . This log page is
global to the controller.
This error log may return the last n errors. If host software specifies a data transfer of the size of n error
logs, then the error logs for the most recent n errors are returned. The ordering of the entries is based on
the time when the error occurred, with the most recent error being returned as the first log  entry.
Each entry in the log page returned is defined in  Figure 205. The log page is a set of 64 -byte entries; the
maximum number of entries supported is indicated in the ELPE field in the Identify Controller data structure
(refer to Figure 312). If the log page is full when a new entry is generated, the controller should insert the
new entry into the log and discard the oldest entry.
The controller should clear this log page by removing all entries on power cycle and Controller Level Reset.
Figure 205: Error Information Log Entry Data Structure
Bytes Description
07:00
Error Count (ECNT): This is a 64-bit incrementing error count, indicating a unique identifier for this
error. The error count starts at 1h, is incremented for each unique error log entry, and is retained
across power off conditions. A value of 0h indicates an invalid entry; this value is used when there
are lost entries or when there a re fewer errors than the maximum number of entries the controller
supports.
If the value of this field is FFFFFFFF_FFFFFFFFh, then the field shall be set to 1h when incremented
(i.e., rolls over to 1h). Prior to NVMe 1.4, processing of incrementing beyond FFFFFFFFh is
unspecified.
09:08
Submission Queue ID  (SQID): This field indicates the Submission Queue Identifier of the
command that the error information is associated with.  If the error is not specific to a particular
command, then this field shall be set to FFFFh.
11:10
Command ID (CID): This field indicates the Command Identifier of the command that the error is
associated with. If the error is not specific to a particular command , then this field shall be set to
FFFFh.
13:12
Status Info (STS):
Bits Description
15:1
Status (STATUS): This field contains the Status field from the completion queue
entry (refer to Figure 100) of the command that completed . If the error is not
specific to a particular command, then this field reports the most applicable status
value.
0 Phase Tag (P): This bit may indicate the Phase Tag posted for the command.

15:14
Parameter Error Location (PEL): This field indicates the byte and bit of the command parameter
that the error is associated with, if applicable. If the parameter spans multiple bytes or bits, then the
location indicates the least-significant byte and bit of the parameter.
Bits Description
15:11 Reserved
10:08 Bit Location (BITLOC): The offset in the byte specified by the Byte Location field
to the bit in that byte that contained the error.
07:00
Byte Location (BYTLOC): The offset in the submission queue entry to the byte
in the command that contained the error. Valid values are based on the SQES
field (refer to Figure 312) (e.g., a value of 6 in SQES indicates that the valid values
for this field are 0 to 63).
If the error is not specific to a particular command, then this field shall be set to FFFFh.
