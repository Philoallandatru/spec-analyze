---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 421
headings: ["5.1.25.1.26.2 Controller Metadata (Feature Identifier 7Eh)"]
---

# Source page 421

NVM Express® Base Specification, Revision 2.1

399
Figure 434: Controller Metadata Element Types
Value Definition
02h Operating System Driver Name: The name of the driver in the operating system as a UTF-8 string.
03h Operating System Driver Version: The version of the driver in the operating system as a UTF-8 string.
04h Pre-boot Controller Name: The name of the controller in the pre-boot environment as a UTF-8 string.
05h Pre-boot Driver Name: The name of the driver in the pre-boot environment as a UTF-8 string.
06h Pre-boot Driver Version: The version of the driver in the pre-boot environment as a UTF-8 string.
07h System Processor Model: The model of the processor as a UTF-8 string.
08h Chipset Driver Name: The chipset driver name as a UTF-8 string.
09h Chipset Driver Version: The chipset driver version as a UTF-8 string.
0Ah Operating System Name and Build: The operating system name and build as a UTF-8 string.
0Bh System Product Name: The system product name as a UTF-8 string.
0Ch Firmware Version: The host firmware (e.g., UEFI) version as a UTF-8 string.
0Dh Operating System Driver Filename: The operating system driver filename as a UTF-8 string.
0Eh Display Driver Name: The display driver name as a UTF-8 string.
0Fh Display Driver Version: The display driver version as a UTF-8 string.
10h
Host-Determined Failure Record: A failure record (e.g., the reason the host has flagged a failure for an
NVMe Storage Device (refer to the NVM Express Management Interface Specification) FRU which may be
used for failure analysis) as a UTF-8 string.
11h to 17h Reserved
18h to 1Fh Vendor Specific
Refer to section 5.1.25.1.26 for the definitions of Command Dword 11 and the Host Metadata data structure.
The default value for the Number of Metadata Element Descriptors of the Enhanced Controller Metadata
Feature shall be 0h on a Controller Level Reset.
If a Get Features command with the SEL field set to 011b (i.e., Supported Capabilities) with the Enhanced
Controller Metadata Feature value is submitted, then the NS Specific bit in Dword 0 of the corresponding
completion queue entry shall be cleared to ‘0’.
 Controller Metadata (Feature Identifier 7Eh)
This Feature is used to store controller metadata about the host platform for later retrieval.
The Controller Metadata Feature provides backward compatibility with Management Controllers (refer to
the NVM Express Management Interface Specification ) compliant with version 1.1 and earlier versions of
the NVM Express Management Interface Specification.
If a controller supports both the Enhanced Controller Metadata Feature and the Controller Metadata
Feature, then the Controller Metadata Feature should not be used by the host.
The metadata element types defined in Figure 434 are used by this Feature.
Refer to section 5.1.25.1.26 for the definitions of Command Dword 11 and the Host Metadata data structure.
If a Set Features command’s Element Action field of Command Dword 11 is set to 10b (Add Entry Multiple),
then the controller shall abort the command with a status code of Invalid Field in Command  and shall not
change the Host Metadata Feature value.
The default value for the Number of Metadata Element Descriptors of the Controller Metadata Feature shall
be 0h on a Controller Level Reset.
If a Get Features command with the SEL field set to 011b (i.e., Supported Capabilities) with the Controller
Metadata Feature value is submitted, then the NS Specific bit in Dword 0 of the corresponding completion
queue entry shall be cleared to ‘0’.
