---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 248
headings: ["5.1.12.1.11 Predictable Latency Per NVM Set (Log Page Identifier 0Ah)"]
---

# Source page 248

NVM Express® Base Specification, Revision 2.1

226
Figure 218: Endurance Group Information Log Page
Bytes Description
63:48
Data Units Read  (DUR): Contains the total number of data bytes that have been read from the
Endurance Group. This value does not include controller reads due to internal operations such as
garbage collection. This value is reported in billions (i.e., a value of 1 corresponds to 1,000,000,000
bytes read) and is rounded up (e.g., one indicates the number of bytes read is from 1 to
1,000,000,000, three indicates the number of bytes read is from 2,000,000,001 to 3,000,000,000).
A value of 0h indicates that the controller does not report the number of Data Units Read.
79:64
Data Units Written (DUW): Contains the total number of data bytes that have been written to the
Endurance Group. This value does not include controller writes due to internal operations such as
garbage collection. This value is reported in billions (i.e., a value of 1 corresponds to 1,000,000,000
bytes written) and is rounded up (e.g., one indicates the number of bytes written is from 1 to
1,000,000,000, three indicates the number of bytes written is from 2,000,000,001 to 3,000,000,000).
A value of 0h indicates that the controller does not report the number of Data Units Written.
95:80
Media Units Written (MUW): Contains the total number of data bytes that have been written to the
Endurance Group including both host and controller writes (e.g., garbage collection). This value is
reported in billions (i.e., a value of 1 corresponds to 1,000,000,000 bytes written) and is rounded up
(e.g., one indicates the number of bytes written is from 1 to 1,000,000,000, three indicates the
number of bytes written is from 2,000,000,001 to 3,000,000,000).
A value of 0h indicates that controller does not report the number of Media Units Written.
111:96
Host Read Commands (HRC): Contains the number of Endurance Group Host Read Commands
completed by the controller.
Refer to the specific I/O Command Set specification for the list of Endurance Group Host Read
Commands that affect this field.
127:112
Host Write Commands (HWC): Contains the number of User Data Out Commands completed by
the controller.
Refer to the specific I/O Command Set specification for the list of User Data Out Commands that
affect this field.
143:128
Media and Data Integrity Errors (MDIE): Contains the number of occurrences where the controller
detected an unrecovered data integrity error for the Endurance Group. Errors such as uncorrectable
ECC, CRC checksum failure, or LBA tag mismatch are included in this field.
159:144 Number of Error Information Log Entries (NEILE): Contains the number of Error Information Log
Entries over the life of the controller for the Endurance Group.
175:160
Total Endurance Group Capacity (TEGCAP):  This field indicates the total NVM capacity in this
Endurance Group. The value is in bytes. If this field is cleared to 0h, then the NVM subsystem does
not report the total NVM capacity in this Endurance Group.
191:176
Unallocated Endurance Group Capacity (UEGCAP):  This field indicates the unallocated NVM
capacity in this Endurance Group. The value is in bytes. If this field is cleared to 0h, then the NVM
subsystem does not report the unallocated NVM capacity in this Endurance Group.
511:192 Reserved
5.1.12.1.11 Predictable Latency Per NVM Set (Log Page Identifier 0Ah)
This log page may be used to determine the current window for the specified NVM Set when Predictable
Latency Mode is enabled and any events that have occurred for the specified NVM Set. There is one log
page for each NVM Set when Predictable Latency Mode is supported. The NVM Set for which the log page
is to be returned is specified in the Log Specific Identifier field in Command Dword 11 of the Get Log Page
command as defined in Figure 219. The log page is 512 bytes in size.
Figure 219: NVM Set Identifier – Log Specific Identifier
Bits Description
15:00 NVM Set Identifier (NVMSETID): This field specifies the identifier for the NVM Set (refer to section
3.2.2) used for this log page.
