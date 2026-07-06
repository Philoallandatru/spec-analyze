---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 213
headings: ["5.1.6.1 Command Completion", "5.1.7 Directive Send command"]
---

# Source page 213

NVM Express® Base Specification, Revision 2.1

191

Figure 176: Directive Receive – Command Dword 10
Bits Description
31:00 Number of Dwords (NUMD): This field specifies the number of dwords to transfer. This is a 0’s based
value.

Figure 177: Directive Receive – Command Dword 11
Bits Description
31:16 Directive Specific (DSPEC): The interpretation of this field is Directive Type dependent. Refer to section
8.1.8.
15:08 Directive Type (DTYPE): This field specifies the Directive Type. Refer to  Figure 597 for the list of
Directive Types.
07:00 Directive Operation (DOPER): This field specifies the Directive Operation to perform. The interpretation
of this field is Directive Type dependent. Refer to section 8.1.8.
 Command Completion
When the command is completed, the controller posts a completion queue entry to the Admin Completion
Queue indicating the status for the command. Command specific status values that may be returned are
dependent on the Directive Type, refer to section 8.1.8.
 Directive Send command
The Directive Send command transfers a data buffer that is dependent on the Directive Type to the
controller. Refer to section 8.1.8.
The Directive Send command uses the Data Pointer, Command Dword 10, and Command Dword 11 fields.
Command Dword 12 and Command Dword 13 may be used based on the Directive Type field and the
Directive Operation field. All other command specific fields are reserved.
Figure 178: Directive Send – Data Pointer
Bits Description
127:00 Data Pointer (DPTR): This field specifies the start of the data buffer. Refer to Figure 92 for the definition
of this field.

Figure 179: Directive Send – Command Dword 10
Bits Description
31:00 Number of Dwords (NUMD): This field specifies the number of dwords to transfer. This is a 0’s based
value.

Figure 180: Directive Send – Command Dword 11
Bits Description
31:16 Directive Specific (DSPEC): The interpretation of this field is Directive Type dependent. Refer to section
8.1.8.
15:08 Directive Type (DTYPE): This field specifies the Directive Type. Refer to  Figure 597 for the list of
Directive Types.
07:00 Directive Operation (DOPER): This field specifies the Directive Operation to perform. The interpretation
of this field is Directive Type dependent. Refer to section 8.1.8.
