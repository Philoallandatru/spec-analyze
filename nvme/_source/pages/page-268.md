---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 268
headings: ["5.1.12.1.14.2.7 Format NVM Start Event (Event Type 07h)", "5.1.12.1.14.2.8 Format NVM Completion Event (Event Type 08h)"]
---

# Source page 268

NVM Express® Base Specification, Revision 2.1

246
Figure 240: Change Namespace Event Data Format (Event Type 06h)
Bytes Description
Notes:
1. For controllers that implement the NVM Command Set, this field contains the value of the associated field from
the Identify Namespace data structure (refer to the NVM Command Set Identify Namespace data structure in
the NVM Command Set Specification).
5.1.12.1.14.2.7 Format NVM Start Event (Event Type 07h)
A Format NVM Start event shall be recorded in the Persistent Event Log after successfully validating the
command parameters of a Format NVM command (refer to section 5.1.10) and before modifying any of the
contents of the NVM.
The Format NVM Start event shall set the Persistent Event Log Event Format Header:
• Event Type field to 07h; and
• Event Type Revision field to 01h.
Figure 241: Format NVM Start Event Data Format (Event Type 07h)
Bytes Description
03:00 Namespace Identifier (NSID): Contains the namespace identifier specified in the Format NVM
command.
04 Format NVM Attributes (FNA): Contains the value from the identify controller FNA field.
07:05 Reserved
11:08 Format NVM CDW10  (FMCDW10): Contains the value from command Dword 10 of the Format
NVM command (refer to Figure 189).
5.1.12.1.14.2.8 Format NVM Completion Event (Event Type 08h)
A Format NVM Completion event shall be recorded in the Persistent Event Log at the completion of a
Format NVM command that resulted in modification of the contents of the NVM.
The Format NVM Completion event shall set the Persistent Event Log Event Format Header:
• Event Type field to 08h; and
• Event Type Revision field to 02h.
Figure 242: Format NVM Completion Event Data Format (Event Type 08h)
Bytes Description
03:00 Namespace Identifier  (NSID): Contains the namespace identifier specified in the Format NVM
command.
04
Smallest Format Progress Indicator  (SFPI): For a Format NVM command that formats a single
namespace, this field contains the smallest numerical value that was available for reporting in the
Remaining Format NVM (RFNVM) field of the FPI field of the I/O Command Set Independent Identify
Namespace data structure (i.e., if the format did not complete successfully and the FPI field is supported,
then this field contains the percentage of the namespace that remained to be formatted at the time the
Format NVM command completed, refer to Figure 319) during the format operation. For a Format NVM
command that formats all namespaces this field shall be cleared to 0h.
05
Format NVM Status (FNVMS): Contains the status of the format operation.
Bits Description
7:2 Reserved
1
Incomplete Format (INCPLTF): If this bit is  set to ‘1’, then the format operation modified
some or all of the user data but did not complete successfully. If this bit is set to ‘1’, then the
Format NVM Error bit shall be set to ‘1’. If this bit is cleared to ‘0’, then the format operation
either did not modify any user data or the format operation completed successfully.
0 Format NVM Error (FNVME): If this bit is set to ‘1’, then the format operation did not complete
successfully. If this bit is cleared to ‘0’, then the format operation completed successfully.
