---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 74
headings: []
---

# Source page 74

NVM Express® Base Specification, Revision 2.1

52
Figure 36: Offset 0h: CAP – Controller Capabilities
Bits Type Reset Description
63:62 RO 0h Reserved
61 RO Impl
Spec
NVM Subsystem Shutdown Enhancements Supported (NSSES): This bit
indicates whether the controller supports enhancements to the NVM Subsystem
Shutdown feature.
If the controller supports the enhancements to the NVM Subsystem Shutdown
feature as defined in section 3.6.3, then this bit shall be set to ‘1’ and the NSSS bit
shall be set to ‘1’. If a controller compliant with a revision of the NVM Express Base
Specification later than revision 2.0 sets the NSSS bit to ‘1’, then that controller
shall set this bit to ‘1’.
If this bit is cleared to ‘0’, then the controller does not support the enhancements to
the NVM Subsystem Shutdown feature as defined in section 3.6.3.
If the NSSRS bit is cleared to ‘0’ or the NSSS bit is cleared to ‘0’, then this bit shall
be cleared to ‘0’.
60:59 RO Impl
Spec
Controller Ready Modes Supported (CRMS): This field indicates the ready
capabilities of the controller. Refer to sections 3.5.3 and 3.5.4 for more detail.
Bits Description
1
Controller Ready Independent of Media Support (CRIMS): If this bit
is set to ‘1’, then the controller supports the Controller Ready
Independent of Media mode.
If this bit is cleared to ‘0’, then the controller does not support Controller
Ready Independent of Media mode.
0
Controller Ready With Media Support (CRWMS): If this bit is set to
‘1’, then the controller supports the Controller Ready With Media mode.
If this bit is cleared to ‘0’, then the controller does not support Controller
Ready With Media mode.
This bit shall be set  to ‘1’ on controllers compliant with NVM Express
Base Specification, Revision 2.0 and later.

58 RO Impl
Spec
NVM Subsystem Shutdown Supported (NSSS):  This bit indicates whether the
controller supports the NVM Subsystem Shutdown feature.
If this bit is set to ‘1’, then the controller supports the NVM Subsystem Shutdown
feature. If the NSSES bit is set to ‘1’, then this bit shall be set to ‘1’.
If this bit is cleared to ‘0’, then the controller does not support the NVM Subsystem
Shutdown feature. If the NSSRS bit is cleared to ‘0’, then this bit shall be cleared
to ‘0’.
Refer to section 3.6.3 for a description of the NVM Subsystem Shutdown feature
and the behavioral enhancements associated with the NSSES bit being set to ‘1’.
57 RO Impl
Spec
Controller Memory Buffer Supported (CMBS): If this bit is set to ‘1’, then this bit
indicates that the controller supports the Controller Memory Buffer, and that
addresses supplied by the host are permitted to reference the Controller Memory
Buffer only if the host has enabled the Controller Memory Buffer’s controller
memory space.
If the controller supports the Controller Memory Buffer, then this bit shall be set to
‘1’.
56 RO Impl
Spec
Persistent Memory Region Supported (PMRS):  This bit indicates whether the
Persistent Memory Region is supported. This bit is set to ‘1’ if the Persistent
Memory Region is supported. This bit is cleared to ‘0’ if the Persistent Memory
Region is not supported.
