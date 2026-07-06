---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 345
headings: []
---

# Source page 345

NVM Express® Base Specification, Revision 2.1

323
Figure 312: Identify – Identify Controller Data Structure, I/O Command Set Independent
Bytes I/O1 Admin1 Disc1 Description
531 M M R
Namespace Write Protection Capabilities (NWPC):  This field indicates the optional
namespace write protection capabilities supported by the controller. Refer to section
8.1.16.
Bits Description
7:3 Reserved
2
Permanent Write Protect Support (PWPS): If this bit is set to ‘1’, then the
controller supports the Permanent Write Protect state. If this bit is cleared to
‘0’, then the controller does not support the Permanent Write Protect state.
If this bit is set to ‘1’, then the controller shall support t he Write Protection
Control field (refer to section 8.1.16.1).
If the NWPWPS bit is cleared to ‘0’, then this bit shall be cleared to ‘0’.
1
Write Protect Until Power Cycle Support (WPUPCS): If this bit is set to
‘1’, then the controller supports the Write Protect Until Power Cycle state. If
this bit is cleared to ‘0’, then the controller does not support Write Protect
Until Power Cycle state. If this bit is set to ‘1’, then the controller shall support
the Write Protection Control field (refer to section 8.1.16.1).
If the NWPWPS bit is cleared to ‘0’, then this bit shall be cleared to ‘0’.
0
No Write Protect and Write Protect Support (NWPWPS) : If this bit is set
to ‘1’, then the controller shall support the No Write Protect and Write Protect
namespace write protection states and may support the Write Protect Until
Power Cycle state and Permanent Write Protect namespace write protection
states (refer to section 8.1.16). If this bit is cleared to ‘0’, then the controller
does not support Namespace Write Protection.

533:532 O R R
Atomic Compare & Write Unit (ACWU): This field is specific to namespaces that are
associated with command sets that specify logical blocks (i.e., Command Set Identifier
0h or 2h), and shall be cleared to 0h for namespaces that are not associated with
command sets that specify logical blocks.  Refer to the applicable I/O Command Set
specification (e.g., the Atomic Operation section of the NVM Command Set Specification).
535:534 M R R
Copy Descriptor Formats Supported (CDFS):
Bits Description
15:5 Reserved
4
Copy Descriptor Format 4 Support (CDF4S): If this bit is set to ‘1’, then the
controller supports Copy Descriptor Format 4h. If this bit is cleared to ‘0’, then
the controller does not support Copy Descriptor Format 4h.
3
Copy Descriptor Format 3 Support (CDF3S): If this bit is set to ‘1’, then the
controller supports Copy Descriptor Format 3h. If this bit is cleared to ‘0’, then
the controller does not support Copy Descriptor Format 3h.
2
Copy Descriptor Format 2 Support (CDF2S): If this bit is set to ‘1’, then the
controller supports Copy Descriptor Format 2h. If this bit is cleared to ‘0’, then
the controller does not support Copy Descriptor Format 2h.
1
Copy Descriptor Format 1 Support (CDF1S): If this bit is set to ‘1’, then the
controller supports Copy Descriptor Format 1h. If this bit is cleared to ‘0’, then
the controller does not support Copy Descriptor Format 1h.
0
Copy Descriptor Format 0 Support (CDF0S): If this bit is set to ‘1’, then the
controller supports Copy Descriptor Format 0h. If this bit is cleared to ‘0’, then
the controller does not support Copy Descriptor Format 0h.
