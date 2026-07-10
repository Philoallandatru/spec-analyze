---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 358
headings: []
---

# Source page 358

NVM Express® Base Specification, Revision 2.1

336
Figure 319: Identify – I/O Command Set Independent Identify Namespace Data Structure
Bytes O/M1 Description Reported2
02 O
Reservation Capabilities (RESCAP):  This field indicates the reservation
capabilities of the namespace. A value of 0h in this field indicates that reservations
are not supported by this namespace. Refer to section 8.1.22 for more details.
Bits Description
7
Ignore Existing Key Support (IEKS): If this bit is set to ‘1’, then the
Ignore Existing Key is used as defined in NVM Express Base
Specification revision 1.3 or later. If this bit is cleared to ‘0’, then the
Ignore Existing Key is used as defined in NVM Express Base
Specification revision 1.2.1 or earlier. This bit shall be set to ‘1’ if the
controller supports revision 1.3 or later a s indicated in the Version
register.
6
Exclusive Access – All Registrants Support (EAARS): If this bit is set
to ‘1’, then  the namespace supports the Exclusive Access – All
Registrants reservation type. If this bit is cleared to ‘0’, then the
namespace does not support the Exclusive Access – All Registrants
reservation type.
5
Write Exclusive – All Registrants Support (WEARS): If this bit is set
to ‘1’, then the namespace supports the Write Exclusive – All Registrants
reservation type. If this bit is cleared to ‘0’, then the namespace does not
support the Write Exclusive – All Registrants reservation type.
4
Exclusive Access – Registrants Only Support (EAROS): If this bit is
set to ‘1’, then  the namespace supports the Exclusive Access –
Registrants Only reservation type. If this bit is cleared to ‘0’, then the
namespace does not support the Exclusive Access – Registrants Only
reservation type.
3
Write Exclusive – Registrants Only Support (WEROS): If this bit is set
to ‘1’, then  the namespace supports the Write Exclusive – Registrants
Only reservation type. If this bit is cleared to ‘0’, then the namespace
does not support the Write Exclusive – Registrants Only reservation type.
2
Exclusive Access Support (EAS): If this bit is set to ‘1’, then  the
namespace supports the Exclusive Access reservation type. If this bit is
cleared to ‘0’, then the namespace does not support the Exclusive
Access reservation type.
1
Write Exclusive Support (WES): If this bit is set to ‘1’, then the
namespace supports the Write Exclusive reservation type. If this bit is
cleared to ‘0’, then the namespace does not support the Write Exclusive
reservation type.
0
Persist Through Power Loss Support (PTPLS): If this bit is set to ‘1’,
then the namespace supports the Persist Through Power Loss
capability. If this bit is cleared to ‘0’, then the namespace does not
support the Persist Through Power Loss capability.

No
