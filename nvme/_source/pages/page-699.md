---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 699
headings: ["B.3 Executing a Fused Operation"]
---

# Source page 699

NVM Express® Base Specification, Revision 2.1

677
Figure 757: PRP List Describing I/O Submission Queue
4 KiB Memory Page i
4 KiB Memory Page j
4 KiB Memory Page k

B.3 Executing a Fused Operation
This example describes how host software creates and executes a fused command, specifically Compare
and Write for a total of 16  KiB of data. In this case, there are two commands that are created. The first
command is the Compare, referred to as CMD0. The s econd command is the Write, referred to as CMD1.
In this case, end-to-end data protection is not enabled and the size of each logical block is 4  KiB.
To build commands for a fused operation, host software utilizes two available adjacent command locations
in the appropriate I/O Submission Queue as is described in section 3.4.2.
The attributes of the Compare command are:
• CMD0.CDW0.OPC is set to 05h for Compare;
• CMD0.CDW0.FUSE is set to 01b indicating that this is the first command of a fused operation;
• CMD0.CDW0.CID is set to a free command identifier;
• CMD0.NSID is set to identify the appropriate namespace;
