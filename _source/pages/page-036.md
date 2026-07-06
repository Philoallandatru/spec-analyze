---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 36
headings: ["1.5.97 Underlying NVM Subsystem", "1.5.98 Underlying Port", "1.5.99 user data", "1.6 I/O Command Set specific definitions used in this specification", "1.6.1 Endurance Group Host Read Command", "1.6.2 Format Index", "1.6.3 SMART Data Units Read Command", "1.6.4 SMART Host Read Command", "1.6.5 User Data Format", "1.6.6 User Data Out Command", "1.7 NVM Command Set specific definitions used in this specification", "1.7.1 logical block", "1.7.2 logical block address (LBA)", "1.8 References"]
---

# Source page 36

NVM Express® Base Specification, Revision 2.1

14
 Underlying NVM Subsystem
Defined as NVM subsystem.
 Underlying Port
A port through which an NVMe subsystem  is attached to a transport (e.g., Ethernet, InfiniBand, Fibre
Channel) (refer to section 1.5.72).
 user data
Data stored in a namespace that is composed of data that the host may store and later retrieve including
metadata if supported.
1.6 I/O Command Set specific definitions used in this specification
The following terms used in this specification are defined in each I/O Command Set specification.
 Endurance Group Host Read Command
An I/O Command Set specific command that results in the controller reading user data, but may or may not
return the data to the host.
 Format Index
A value used to index into the I/O Command Set Specific Format table (i.e., the User Data Format number).
 SMART Data Units Read Command
An I/O Command Set specific command that results in the controller reading user data, but may or may not
return the data to the host.
 SMART Host Read Command
An I/O Command Set specific command that results in the controller reading user data, but may or may not
return the data to the host.
 User Data Format
An I/O Command Set specific format that describes the layout of the data on the NVM media.
 User Data Out Command
An I/O Command Set specific command that results in the controller writing user data, but may or may not
transfer user data from the host to the controller.
1.7 NVM Command Set specific definitions used in this specification
The following terms used in this specification are defined in the NVM Command Set Specification. These
terms are used throughout the document as examples for a specific I/O Command Set.
 logical block
The smallest addressable data unit for Read and Write commands.
 logical block address (LBA)
The address of a logical block, referred to commonly as LBA.
1.8 References
CNSA 1.0, “USE OF PUBLIC STANDARDS FOR SECURE INFORMATION SHARING”, CNSSP 15
ANNEX B “NSA -APPROVED COMMERCIAL NATIONAL SECURITY ALGORITHM (CNSA) SUITE”, 20
October 2016. Available from https://www.cnss.gov/CNSS/issuances/Policies.cfm.
