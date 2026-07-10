---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 548
headings: ["8.1.13 Key Per I/O"]
---

# Source page 548

NVM Express® Base Specification, Revision 2.1

526
a. the Select field set to the Get Controller State management operation (i.e., 0h);
b. the Sequence Indicator field set to the proper sequence value;
c. the Controller Identifier field set to the controller identifier for the Migratable Controller in the
Source NVM Subsystem;
d. the Controller State Version Index field set to an index into the NVMe Controller State Version
list in the Supported Controller State Formats data structure for MMC in the Source NVM
Subsystem (refer to Figure 329) if the NVMe defined controller state is required; and
e. the Controller State UUID Index field set to an index in the Vendor Specific Controller State
UUID Supported list in the Supported Controller State Formats Data Structure for MMC in the
Source NVM Subsystem (refer to Figure 329), if vendor specific controller state is required.
7. After the MMHS has transferred the obtained controller state information to the MMHD, the MMHD
sets that controller state into a Migratable Controller by submitting a sequence of one or more
Migration Send commands to the MMC in the Destination NVM Subsystem specifying:
• the Select field set to Set Controller State management operation (i.e., 3h);
• the Sequence Indicator field set to the proper sequence value;
• the Controller Identifier field set to the controller identifier for the Migratable Controller in
the Destination NVM Subsystem;
• the Controller State Version Index field set to an index into the NVMe Controller State
Version list in the Supported Controller State Formats data structure for MMC in the
Destination NVM Subsystem (refer to Figure 329) if value of the NVMe Controller State
Size (NVMECSS) field is non-zero (refer to Figure 358); and
• the Controller State UUID Index field set to an index in the Vendor Specific Controller State
UUID Supported list in the Supported Controller State Formats Data Structure for MMC in
the Source NVM Subsystem (refer to Figure 329), if the value of the Vendor Specific Size
(VSS) field is non-zero (refer to Figure 358).
8. The MMHD resumes operation on the Migratable Controller  in the Destination NVM Subsystem
(i.e., the migrated controller) by submitting a Migration Send command to the MMC in the
Destination NVM Subsystem specifying:
• the Select field set to Resume management operation (i.e., 2h);
• the Controller Identifier field (refer to Figure 166) set to the controller identifier for the
Migratable Controller to be resumed.
 Key Per I/O
The Key Per I/O capability provides a mechanism to use encryption keys that have been injected into an
NVM subsystem by a host. The mechanism to perform activation of the Key Per I/O capability, encryption
key injection, management, and association to encryption key tag is outside the scope of this specification.
One mechanism is defined in the TCG Storage Security Subsystem Class: Key Per I/O Specification using
the NVMe Admin commands Security Send and Security Receive. If the TCG mechanism is used, any
additional modifications to the NVM subsystem as a result of activation of the Key Per I/O Security Provider
are defined in the TCG Storage Interface Interactions Specification (SIIS).
Encryption keys injected into the NVM subsystem may be referenced by I/O commands through the use of
the encryption key tag (refer to the KEYTAG field in Figure 620) associated with the encryption keys. If an
I/O command CETYPE field is set to the KPIOTAG value, then the CEV field (refer to Figure 620) of that
I/O command specifies the encryption key tag associated with the encryption key to be used to encrypt or
decrypt the data in that I/O command. The association of an encryption key tag to a specific encryption key
is outside the scope of this specification. One association mechanism is defined in the TCG Storage
Security Subsystem Class: Key Per I/O Specification.
The Key Per I/O Scope bit (refer to Figure 312) indicates if the Key Per I/O capability:
• applies to all read and write commands in all namespaces within the NVM subsystem; or
• independently applies to read and write commands in each namespace within the NVM subsystem.
