---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 338
headings: []
---

# Source page 338

NVM Express® Base Specification, Revision 2.1

316
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
343 O O R
Asymmetric Namespace Access Capabilities (ANACAP):  This field indicates the
capabilities associated with Asymmetric Namespace Access Reporting (refer to section
8.1.1).
Bits Description
7
ANA Group ID Support (ANAGIDS): Iif this bit is  set to ‘1’, then the
controller supports a non -zero value in the ANAGRPID field of the
Namespace Management command. If this bit is  cleared to ‘0’, then the
controller does not support a non -zero value in the ANAGRPID field of the
Namespace Management command. If the Namespace Management
command is not supported, then this bit shall be cleared to ‘0’.
6
ANA Group ID Locked When Attached Support (ANAGIDLWAS): Iif this
bit is set to ‘1’, then the ANAGRPID field in the Identify Namespace data
structure (refer to the NVM Command Set Specification) does not change
while the namespace is attached to any controller. If this bit is cleared to ‘0’,
then the ANAGRPID field may change while the namespace is attached to
any controller. Refer to section 8.1.1.2.
5 Reserved
4
Report ANA Change State (RANACS) :  Iif this bit is set to ‘1’, then the
controller is able to report ANA Change state (refer to section 8.1.1.8). If this
bit is cleared to ‘0’, then the controller does not report ANA Change state.
3
Report ANA Persistent Loss State (RANAPLS): Iif this bit is set to ‘1’, then
the controller is able to report ANA Persistent Loss state (refer to section
8.1.1.7). If this bit is cleared to ‘0’, then the controller does not report ANA
Persistent Loss state.
2
Report ANA Inaccessible State (RANAIS): Iif this bit is set to ‘1’, then the
controller is able to report ANA Inaccessible state (refer to section 8.1.1.6).
If this bit is cleared to ‘0’, then the controller does not report ANA Inaccessible
state.
1
Report ANA Non-Optimized State (RANANOS): Iif this bit is set to ‘1’, then
the controller is able to report ANA Non -Optimized state (refer to section
8.1.1.5). If this bit is cleared to ‘0’, then the controller does not report ANA
Non-Optimized state.
0
Report ANA Optimized State (RANAOS):  Iif this bit is  set to ‘1’, then the
controller is able to report ANA Inaccessible state (refer to section 8.1.1.4).
If this bit is cleared to ‘0’, then the controller does not report ANA Inaccessible
state.

347:344 O O R
ANA Group Identifier Maximum (ANAGRPMAX):  This field indicates the maximum
value of a valid ANA Group Identifier for any controller in the NVM subsystem. If the
controller supports Asymmetric Namespace Access Reporting (refer to the CMIC field in
Figure 312), then this field shall be set to a non -zero value. If the controller does not
support Asymmetric Namespace Access Reporting, then this field shall be cleared to 0h.
351:348 O O R
Number of ANA Group Identifiers (NANAGRPID):  This field indicates the number of
ANA groups supported by the controller. If the controller supports Asymmetric
Namespace Access Reporting (refer to the CMIC field in Figure 312), then this field shall
be set to a non -zero value that is less than or equal to the ANAGRPMAX value. If the
controller does not support Asymmetric Namespace Access Reporting, then this field
shall be cleared to 0h.
355:352 O O R
Persistent Event Log Size (PELS): This field indicates the maximum reportable size for
the Persistent Event Log (Refer to section 5.1.12.1.14) in 64 KiB units. If the Persistent
Event Log is not supported, then this field is reserved.
357:356 O O O
Domain Identifier (DID): This field indicates the identifier of the domain (refer to section
3.2.5.3) that contains this controller. If the MDS bit is set to ‘1’, then this field shall be set
to a non-zero value. If the NVM subsystem does not support multiple domains (i.e., the
NVM subsystem consists of a single domain), then this field shall be cleared to 0h.
