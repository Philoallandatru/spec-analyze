---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 158
headings: []
---

# Source page 158

NVM Express® Base Specification, Revision 2.1

136
Figure 92: Common Command Format
Bytes Description
39:24
Data Pointer (DPTR): This field specifies the data used in the command.
If CDW0.PSDT is cleared to 00b, then the definition of this field is:
39:32
PRP Entry 2 (PRP2): This field:
• is reserved if the data transfer does not cross a memory page boundary;
• specifies the Page Base Address of the second memory page if the data
transfer crosses exactly one memory page boundary. E.g.,:
o the command data transfer length is equal in size to one memory
page and the offset portion of the PBAO field of PRP1 is non -
zero; or
o the Offset portion of the PBAO field of PRP1 is equal to 0h and
the command data transfer length is greater than one memory
page and less than or equal to two memory pages in size;
and
• is a PRP List pointer if the data transfer crosses more than one memory
page boundary. E.g.,:
o the command data transfer length is greater than or equal to two
memory pages in size but the offset portion of the PBAO field of
PRP1 is non-zero; or
o the command data transfer length is equal in size to more than
two memory pages and the Offset portion of the PBAO field of
PRP1 is equal to 0h.
31:24
PRP Entry 1 (PRP1): This field contains:
• the first PRP entry for the command; or
• a PRP List pointer,
depending on the command  (e.g., the Create I/O Completion Queue command
(refer to Figure 473)).
If CDW0.PSDT is set to 01b or 10b, then the definition of this field is:
39:24
SGL Entry 1 (SGL1): This field contains the first SGL segment for the command.
If the SGL segment is an SGL Data Block or Keyed SGL Data Block or Transport
SGL Data Block descriptor, then it describes the entire data transfer. If more than
one SGL segment is needed to describ e the data transfer, then the first SGL
segment is a Segment, or Last Segment descriptor. Refer to section 4.3.2 for the
definition of SGL segments and descriptor types.
The NVMe Transport may support a subset of SGL Descriptor types and features
as defined in the NVMe Transport binding specification.

43:40 Command Dword 10 (CDW10): This field is command specific Dword 10.
47:44 Command Dword 11 (CDW11): This field is command specific Dword 11.
51:48 Command Dword 12 (CDW12): This field is command specific Dword 12.
55:52 Command Dword 13 (CDW13): This field is command specific Dword 13.
59:56 Command Dword 14 (CDW14): This field is command specific Dword 14.
63:60 Command Dword 15 (CDW15): This field is command specific Dword 15.
In addition to the fields commonly defined for the Common Command Format, Vendor Specific commands
may support the Number of Dwords in Data Transfer and Number of Dwords in Metadata Transfer fields. If
supported, the command format for the Vendor Specific commands are defined in Figure 93. For more
details, refer to section 8.1.26.
