---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 572
headings: []
---

# Source page 572

NVM Express® Base Specification, Revision 2.1

550
Figure 634: RPMB Request and Response Message Types
Request Message Types Description Requires
Data
RPMB Frame
Length
(bytes)
0200h Reading of the Write
Counter value response
Returned as a result of the host requesting a Result
read request Message Type after requesting the
Write Counter value.
No 256
0300h Authenticated data write
response
Returned as a result of the host requesting a Result
read request Message Type after attempting to write
data to an RPMB target.
No 256
0400h Authenticated data read
response
Returned as a result of the host requesting a Result
read request Message Type after attempting to read
data from an RPMB target.
Yes M + 256
0600h
Authenticated Device
Configuration data write
response
Returned as a result of the host requesting a Result
read request Message Type after attempting to write
a Device Configuration Block to an RPMB target.
No 256
0700h
Authenticated Device
Configuration data read
response
Returned as a result of the host requesting a Result
read request Message Type after attempting to read
DCB from an RPMB target.
Yes 512 + 256
The operation result defined in Figure 635 indicates whether an RPMB request was successful or not.
Figure 635: RPMB Operation Result
Bits Description
15:08 Reserved
07 Write Counter Status (WCS): Indicates if the Write Counter has expired (i.e., reached its maximum value).
A value of ‘1’ indicates that the Write Counter has expired. A value of ‘0’ indicates a valid Write Counter.
06:00
Operation Status (OPSTAT): Indicates the operation status. Valid operation status values are listed below.
Value Definition
00h Operation successful
01h General failure
02h Authentication failure (MAC comparison not matching, MAC calculation failure)
03h Counter failure (counters not matching in comparison, counter incrementing failure)
04h Address failure (address out of range, wrong address alignment)
05h Write failure (data/counter/result write failure)
06h Read failure (data/counter/result read failure)
07h Authentication Key not yet programmed
08h Invalid RPMB Device Configuration Block – this may be used when the target is not 0h.
09 to 3Fh Reserved

Figure 636 defines the non-volatile contents stored within the controller for each RPMB target.
Figure 636: RPMB Contents
Content Type Size Description
Authentication
Key
Write once,
not erasable
or readable
Size is dependent on
authentication method
reported in Identify
Controller data structure
(e.g., SHA-256 is 32 bytes
(refer to RFC 6234))
Authentication key which is used to authenticate
accesses when MAC is calculated.
