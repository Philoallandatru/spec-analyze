---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 422
headings: ["5.1.25.1.26.3 Namespace Metadata (Feature Identifier 7Fh)", "5.1.25.1.27 Software Progress Marker (Feature Identifier 80h)", "5.1.25.1.28 Host Identifier (Feature Identifier 81h)"]
---

# Source page 422

NVM Express® Base Specification, Revision 2.1

400
 Namespace Metadata (Feature Identifier 7Fh)
This Feature is used to store metadata about a namespace associated with a controller for later retrieval.
This Feature is namespace specific  and controller specific. The Add Entry Multiple action is prohibited for
this Feature.
Refer to section 5.1.25.1.26 for the definitions of Command Dword 11 and the Host Metadata data structure.
If a Get Features command with the SEL field set to 011b (i.e., Supported Capabilities) with the Namespace
Metadata Feature value is submitted, then the NS Specific bit in Dword 0 of the corresponding completion
queue entry shall be set to ‘1’.
5.1.25.1.27 Software Progress Marker (Feature Identifier 80h)
This Feature is a controller software progress marker. The software progress marker is persistent across
power states. This information may be used to indicate to an OS software driver whether there have been
issues with the OS successfully loading. The attributes are specified in Command Dword 11.
If a Get Features command is submitted for this Feature, the attributes specified in Figure 436 are returned
in Dword 0 of the completion queue entry for that command.
Figure 436: Software Progress Marker – Command Dword 11
Bits Description
31:08 Reserved
07:00
Pre-boot Software Load Count (PBSLC): Indicates the load count of pre -boot software.  After
successfully loading and initializing the controller, p re-boot software should set this field to one more
than the previous value of the Pre-boot Software Load Count. If the previous value is 255, then the value
should not be updated by pre-boot software (i.e., the value does not wrap to 0). OS driver software should
set this field to 0h after the OS has successfully been initialized.
5.1.25.1.28 Host Identifier (Feature Identifier 81h)
This Feature allows the host to register a Host Identifier with the controller. The Host Identifier is used by
the controller to determine whethe r other controllers in the NVM s ubsystem are associated with the same
host. The Host Identifier may be used to designate host elements that access an NVM subsystem
independently of each other or for reservations.
The Host Identifier is contained in the data structure indicated in Figure 438. The attributes are specified in
Command Dword 11. If a Get Features command is issued for this Feature, the data structure specified in
Figure 438 is returned in the data buffer for that command.
A Host Identifier value of 0h indicates that the host associated with the controller is not associated with any
other controller in the NVM subsystem. For example, two controllers in an NVM subsystem that both have
a Host Identifier value of 0h indicates t hat the controllers are associated with different hosts. NVMe over
PCIe implementations may support using a Host Identifier value of 0h for the reservations feature (refer to
Figure 435: Namespace Metadata Element Types
Value Definition
00h Reserved
01h Operating System Namespace Name: The name of the namespace in the operating system as a
UTF-8 string.
02h Pre-boot Namespace Name: The name of the namespace in the pre-boot environment as a UTF-8
string.
03h Operating System Namespace Name Qualifier 1: The first qualifier of the Operating System
Namespace Name as a UTF-8 string.
04h Operating System Namespace Name Qualifier 2 : The second qualifier of the Operating System
Namespace Name as a UTF-8 string.
05h to 17h Reserved
18h to 1Fh Vendor Specific
