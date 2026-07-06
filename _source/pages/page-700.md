---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 700
headings: []
---

# Source page 700

NVM Express® Base Specification, Revision 2.1

678
• If metadata is being used in a separate buffer, then the location of that buffer is specified in the
CMD0.MPTR field;
• The physical address of the first page of the data to compare:
o If PRPs are used, CMD0.PRP1 is set to the physical address of the first page of the data
to compare and CMD0.PRP2 is set to the physical address of the PRP List. The PRP List
is shown in Figure 758 for a PRP List with three entries; or
o If the command uses SGLs, CMD0.SGL1 is set to an appropriate SGL segment descriptor
depending on whether more than one descriptor is needed;
• CMD0.CDW10.SLBA is set to the first LBA to compare against. Note that this field also spans
Command Dword 11;
• CMD0.CDW12.LR is cleared to ‘0’ to indicate that the controller should apply all available error
recovery means to retrieve the data for comparison;
• CMD0.CDW12.FUA is cleared to ‘0’, indicating that the data may be read from any location,
including a volatile cache, in the NVM subsystem;
• CMD0.CDW12.PRINFO is cleared to 0h since end-to-end protection is not enabled;
• CMD0.CDW12.NLB is set to 3h, indicating that four logical blocks of a size of 4 KiB each are to be
compared against;
• CMD0.CDW14 is cleared to 0h since end-to-end protection is not enabled; and
• CMD0.CDW15 is cleared to 0h since end-to-end protection is not enabled.
Figure 758: PRP List Describing Data to Compare
LBA #0 (4 KiB)
LBA #1 (4 KiB)
LBA #2 (4 KiB)
LBA #3 (4 KiB)

The attributes of the Write command are:
• CMD1.CDW0.OPC is set to 01h for Write;
• CMD1.CDW0.FUSE is set to 10b indicating that this is the second command of a fused operation;
• CMD1.CDW0.CID is set to a free command identifier;
