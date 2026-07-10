---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 592
headings: []
---

# Source page 592

NVM Express® Base Specification, Revision 2.1

570
Deallocate After Sanitize (NDAS) bit set to ‘1’ in the Sanitize command that started the sanitize
operation;
• the No-Deallocate Inhibited (NDI) bit, which indicates if the controller supports the No -Deallocate
After Sanitize bit in the Sanitize command; and
• the Verification Support (VERS) bit, which indicates if the controller supports the Media Verification
state and the Post -Verification Deallocation state for sanit ization operations that perform  block
erase or crypto erase.
If the NODMMAS field indicates a value of 10b in the Identify Controller data structure (refer to Figure 312)
and a Sanitize command that starts a sanitize operation specifies the No-Deallocate After Sanitize (NDAS)
bit set to ‘1’, then sanitize processing includes additional media modification. Refer to Annex A.3 for further
information about sanitize operations and interactions with integrity circuits.
The Overwrite sanitize operation is media specific and may not be appropriate for all media types. For
example, if the media is NAND, multiple pass overwrite operations may have an adverse effect on media
endurance.
If the NVM subsystem supports the Key Per I/O capability (refer to section 8.1.11), then a sanitize operation
shall alter all user data such that recovery of any previous user data using the KEYTAG values specified
when that previous user data was written (i.e., original KEYTAG values) is infeasible for a given level of
effort (refer to ISO/IEC 27040).
Figure 647: Sanitize Operations – Overwrite Mechanism
OIPBP1
Overwrite
Pass Count1
Overwrite
Pass Number User Data except PI Metadata Protection Information2
‘0’ All All Overwrite Pattern1 Each byte set to FFh
‘1’ Even
First Inversion of Overwrite Pattern1 Each byte cleared to 00h
Subsequent Inversion of Overwrite Pattern1 from previous pass (i.e., each bit
XORed with ‘1’)
‘1’ Odd
First Overwrite Pattern1 Each byte set to FFh
Subsequent Inversion of Overwrite Pattern 1 from previous pass (i.e., each bit
XORed with ‘1’)
Notes:
1. Parameters are specified in Command Dword 10 and Command Dword 11 of the corresponding Sanitize
command that started the Overwrite operation. The Overwrite Invert Pattern Between Passes (OIPBP) field is
defined in Command Dword 10. The Overwrite Pass Count  field is defined in Command Dword 10. The
Overwrite Pattern field is defined in Command Dword 11. Refer to section 5.1.22.
2. If Protection Information is present within the metadata.
To start a sanitize operation, the host submits a Sanitize command specifying the SANACT field set to:
• 010b (i.e., start a Block Erase type sanitize operation);
• 011b (i.e., start a Overwrite type sanitize operation); or
• 100b (i.e., start a Crypto Erase type sanitize operation).
The Sanitize command specifies other parameters, including:
• the Allow Unrestricted Sanitize Exit (AUSE) bit;
• the No-Deallocate After Sanitize (NDAS) bit; and
• the Enter Media Verification State (EMVS) bit.
After validating the Sanitize command parameters, the controller starts the sanitize operation in the
background, updates the Sanitize Status log page and then completes the Sanitize command with
Successful Completion status.
