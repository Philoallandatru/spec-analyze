---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 100
headings: []
---

# Source page 100

NVM Express® Base Specification, Revision 2.1

78
Figure 67: NVM Sets and Associated Namespaces

There is a subset of Admin commands that are NVM Set aware as described in Figure 68.
Figure 68: NVM Set Aware Admin Commands
Admin Command Details
Identify
• The Identify Namespace data structure includes the associated NVM Set
Identifier.
• The NVM Set List data structure includes attributes for each NVM Set.
Capacity Management
• The Create NVM Set action returns the NVM Set Identifier of the NVM Set that
is created.
• The Delete NVM Set action includes the NVM Set Identifier of the NVM Set that
is to be deleted.
Namespace Management • The create action includes the NVM Set Identifier as a host specified field.
Get Features and
Set Features
• The Read Recovery Level Feature specifies the associated NVM Set Identifier.
• The Predictable Latency Mode Config Feature specifies the associated NVM Set
Identifier.
• The Predictable Latency Mode Window Feature specifies the associated NVM
Set Identifier.
Connect • The Connect command includes the associated NVM Set Identifier.
Create I/O Submission Queue • The Create I/O Submission Queue command includes the associated NVM Set
Identifier.
Get Log Page • The Predictable Latency Per NVM Set log page specifies the associated NVM
Set Identifier.
The host determines the NVM Sets present and their attributes using the Identify command with CNS value
of 04h to retrieve the NVM Set List (refer to Figure 317). For each NVM Set, the attributes include:
• an identifier associated with the NVM Set;
• the optimal size for writes to the NVM Set;
