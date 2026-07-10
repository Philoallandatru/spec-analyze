---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 220
headings: ["5.1.10.1 Command Completion", "5.1.11 Get Features command"]
---

# Source page 220

NVM Express® Base Specification, Revision 2.1

198
Figure 189: Format NVM – Command Dword 10
Bits Description
11:09
Secure Erase Settings (SES): This field specifies whether a secure erase should be performed as part
of the format and the type of the secure erase operation. The erase applies to all user data, regardless
of location (e.g., within an exposed LBA  (refer to the NVM Express NVM Command Set Specification),
within a cache, within deallocated logical blocks, etc.).
Value Definition
000b No secure erase operation requested
001b
User Data Erase: All user data shall be erased, contents of the media allocated for
user data after the erase is indeterminate (e.g., the user data may be zero filled, one
filled, etc.). If a User Data Erase is requested and all affected user data is encrypted,
then the controller is allowed to use a cryptographic erase to perform the requested
User Data Erase.
010b Cryptographic Erase:  All user data shall be erased cryptographically. This is
accomplished by deleting the encryption key.
011b to 111b Reserved

08 Protection Information Location (PIL): I/O Command Set specific definition.1
07:05 Protection Information (PI): I/O Command Set specific definition.1
04 Metadata Settings (MSET): I/O Command Set specific definition.1
03:00
LBA Format Lower (LBAFL): This field specifies the least significant 4 bits of the Format Index of the
User Data Format to apply to the NVM media. This corresponds to the User Data Formats indicated in
the Identify Namespace data structure, refer to the Identify Namespace data structure  and the I/O
Command Set specific Format data structure in the applicable I/O Command Set specification. If an
unsupported User Data Format is selected, the controller shall abort the command with a status code of
Invalid Format.
Note: This field applies to all User Data Formats. The original name has been retained for historical
continuity.
Note:
1. I/O Command Set specific fields are described in the applicable I/O Command Set specification.
 Command Completion
A completion queue entry is posted to the Admin Completion Queue when the NVM media format is
complete. Format NVM command specific status values (i.e., SCT field set to 1h) are shown in Figure 190.
Figure 190: Format NVM – Command Specific Status Values
Value Description
0Ah
Invalid Format: The format specified is invalid. This may be due to various conditions, including:
• specifying an invalid User Data Format number;
• enabling protection information when there are not sufficient metadata resources; or
• the specified format is not available in the current configuration.
 Get Features command
The Get Features command retrieves the attributes of the Feature specified.
The Get Features command uses the Data Pointer, Command Dword 10 and Command Dword 14 fields.
The use of the Command Dword 11 field is Feature specific. If not used by a Feature, then Command
Dword 11 is reserved unless otherwise stated. All other command specific fields are reserved.
The mandatory, optional, and prohibited Feature Identifiers for each type of controller are defined in section
3.1.3.6.
