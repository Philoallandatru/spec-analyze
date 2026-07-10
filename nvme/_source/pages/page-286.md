---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 286
headings: ["5.1.12.1.21 Boot Partition (Log Page Identifier 15h)"]
---

# Source page 286

NVM Express® Base Specification, Revision 2.1

264
Figure 266: Command and Feature Lockdown Log Page
Bytes Description
0
Command and Feature Identifier List Attributes (CFILA): This field indicates the contents of the
Command and Feature Identifier List field in the log page.
Bits Description
7:6 Reserved
5:4
Contents Selected (CS) : This field in combination with the Scope Selected field
indicates the contents of the Command and Feature Identifier List field in the log page.
The Content Selected field is specified by the contents of the Contents field in the Log
Specific Parameter field of the Get Log Page command.
Value Description
00b List contains command opcodes or Set Features Feature Identifiers based
on the Scope Selected field that are supported to be prohibited
01b
List contains command opcodes or Set Features Feature Identifiers based
on the Scope Selected field that are currently prohibited if received o n an
Admin submission queue
10b
List contains command opcodes or Set Features Feature Identifiers based
on the Scope field that are currently prohibited if received out-of-band on a
Management Endpoint
11b Reserved

3:0
Scope Selected (SS): This field in combination with the Contents Selected field
indicates what the Command and Feature Identifier List field contains in the log page.
The Scope Selected field is specified by the contents of the Scope field in the Log
Specific Parameter field of the Get Log Page command.
Value Definition
0h List contains Admin Command Set opcodes
1h Reserved
2h List contains Set Features Feature Identifiers
3h List contains Management Interface Command Set opcodes
4h List contains PCIe Command Set opcodes
5h to Fh Reserved


2:1 Reserved
3
Length (LNGTH): This field indicates the length in bytes (n) of the Command and Feature Identifier
List field that follow in the log page. If the Command and Feature Identifier List field contains no
coded values, then this field shall be cleared to 0h.
n+3:4
Command and Feature Identifier List (CFIL) : The contents of this field are dependent on the
setting of the Contents Selected field and Scope Selected  field. This field contains a list of coded
values identified by the Scope Selected field and the Content Selected field. The list shall be in order
from lowest numerical value to highest numerical value.
511:n+4 Reserved
5.1.12.1.21 Boot Partition (Log Page Identifier 15h)
The Boot Partition log page provides read only access to the Boot Partition (refer to section 8.1.3)
accessible by this controller through the BPRSEL register (refer to section 3.1.4.14).
This log page consists of a header describing the Boot Partition and Boot Partition data as defined by Figure
268. The Boot Partition Identifier bit in the Log Specific Parameter field determines the Boot Partition.
A host reading this log page has no effects on the BPINFO (refer to section 3.1.4.13), BPRSEL, and BPMBL
(refer to section 3.1.4.15) registers.
The Log Specific Parameter field in Command Dword 10 (refer to Figure 197) for this log page is defined
in Figure 267.
