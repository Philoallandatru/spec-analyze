---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 697
headings: ["A.4 Bad Media and Vendor Specific NAND Use"]
---

# Source page 697

NVM Express® Base Specification, Revision 2.1

675
A.4 Bad Media and Vendor Specific NAND Use
Another audit capability that is not supported by NVM Express is checking that any media that could not be
sanitized (e.g., bad physical blocks) has been removed from the pool of storage that is able to be used as
addressable storage.
An approach that is performed under some circumstances is removing the physical storage components
from the NVM Express device after a sanitize operation and reading the contents in laboratory conditions.
However, this approach also has multiple difficulties. When physical storage components are removed from
an NVM Express device, much context is lost. This includes:
a) any encoding for zero’s/one’s balance;
b) identification of which components contain device firmware or other non-data information; and
c) which media has been retired and cannot be sanitized.
