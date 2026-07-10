---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 398
headings: ["5.1.25.1.4 Volatile Write Cache (Feature Identifier 06h)"]
---

# Source page 398

NVM Express® Base Specification, Revision 2.1

376
Figure 389: Temperature Threshold – Command Dword 11
Bits Description
31:25 Reserved
24:22
Temperature Threshold Hysteresis (TMPTHH) : This field indicates the temperature hysteresis (i.e.,
the number of kelvins the controller uses to determine the end of the over temperature or under
temperature event). If this field is cleared to 000b, then the hysteresis as defined for this Feature does
not apply.
If the Temperature Threshold Maximum Hysteresis (TMPTHMH) field in Figure 312 is cleared to 0h, this
field shall be cleared to 000b.
21:20
Threshold Type Select (THSEL): This field selects the threshold type that is modified by a Set Features
command and whose threshold value is returned by a Get Features command.
Value Definition
00b Over Temperature Threshold
01b Under Temperature Threshold
10b to 11b Reserved

19:16
Threshold Temperature Select (TMPSEL):  This field selects the temperature whose threshold is
modified by a Set Features command and whose threshold value is returned by a Get Features
command.
Value Definition
0h Composite Temperature
1h Temperature Sensor 1
2h Temperature Sensor 2
3h Temperature Sensor 3
4h Temperature Sensor 4
5h Temperature Sensor 5
6h Temperature Sensor 6
7h Temperature Sensor 7
8h Temperature Sensor 8
9h to Eh Reserved
Fh All implemented temperature sensors in a Set Features
command. Reserved in a Get Features command.

15:00 Temperature Threshold (TMPTH): Indicates the threshold value for the temperature sensor and
threshold type specified in Kelvins.
5.1.25.1.4 Volatile Write Cache (Feature Identifier 06h)
This Feature controls the volatile write cache, if present, on the controller. If a volatile write cache is present
(refer to the VWC field in Figure 312), then this Feature shall be supported. The attributes are specified in
Command Dword 11.
Note: If the controller is able to guarantee that data present in a write cache is written to non-volatile storage
media on loss of power, then that write cache is considered non-volatile and this Feature does not apply to
that write cache.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 390 are returned
in Dword 0 of the completion queue entry for that command.
If a volatile write cache is not present, then a Set Features command specifying the Volatile Write Cache
feature identifier shall abort with a status code of Invalid Field in Command, and a Get Features command
specifying the Volatile Write Cache feature identifier shall abort with a status code of Invalid Field in
Command.
If a volatile write cache is present and the volatile write cache is disabled (i.e., the WCE bit is cleared to
‘0’), then the user data written by any command to a namespace shall be persistent. Refer to the Volatile
Write Cache Presence field contained in the Common Namespace Features field in Figure 319.
