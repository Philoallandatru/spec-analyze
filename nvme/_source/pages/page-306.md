---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 306
headings: ["5.1.12.2 Memory-Based Log Specific Information (PCIe)", "5.1.12.3 Message-Based Log Specific Information (Fabrics)", "5.1.12.3.1 Discovery (Log Page Identifier 70h)"]
---

# Source page 306

NVM Express® Base Specification, Revision 2.1

284
 Memory-Based Log Specific Information (PCIe)
There are no log pages that are specific to the Memory-based transport model.
 Message-Based Log Specific Information (Fabrics)
This section describes log pages that are specific to the Message-based transport model.
5.1.12.3.1 Discovery (Log Page Identifier 70h)
The Discovery log page shall only be supported by Discovery controllers. The Discovery log page shall not
be supported by controllers that expose namespaces for NVMe over PCIe or NVMe over Fabrics. The
Discovery log page provides an inventory of NVM subsys tems with which a host may attempt to form an
association. Depending on the parameters specified in a Get Log Page command, the Discovery log page
may contain entries for:
• all known NVM subsystems; or
• a subset of entries specific to the host or Discovery controller requesting the log page.
The Discovery log page is persistent across power cycles.
Figure 292 specifies the format for the LID Specific Parameter field in the Supported Log Pages log page
(refer to section 5.1.12.1.1) for the Discovery log page.
Figure 292: Discovery Log Page – LID Specific Parameter Field
Bits Description
15:3 Reserved
2
All NVM Subsystem Entries Supported (ALLSUBES): If this bit is set to ‘1’, then the Discovery controller
supports returning records for all NVM subsystem ports that are registered without filtering the list of
returned records based upon the requesting HOSTNQN. If this bit is cleared to ‘0’, then the Discovery
controller does not support returning records for all NVM subsystem ports that are registered.
1
Port Local Entries Only Supported (PLEOS): If this bit is set to ‘1’, then the Discovery controller supports
returning only port local Discovery Log Page Entries or port local Extended Discovery Log Page Entries. If
this bit is cleared to ‘0’, then the Discovery controller does not support returning only port local Discovery
Log Page Entries or port local Extended Discovery Log Page Entries. If the Discovery controller is not a
DDC, then this bit should be ignored.
0
Extended Discovery Log Page Entry Supported (EXTDLPES): If this bit is set to ‘1’, then the Discovery
controller supports returning Extended Discovery Log Page Entries. If this bit is cleared to ‘0’, then the
Discovery controller does not support returning Extended Discovery Log Page Entries. If the Discovery
controller is a CDC, then that CDC shall support returning Extended Discovery Log Page Entries.
If a Discovery controller supports returning Extended Discovery Log Page Entries (i.e., the Extended
Discovery Log Page Entries Supported (EXTDLPES) bit is set to ‘1’ in the LID Specific Parameter field as
defined in Figure 292) and that Discovery controller processes a Get Log Page command for this log page
with the Extended Discovery Log Page Entries (EXTDLPE) bit set to ‘1’ in the Log Specific Parameter (LSP)
field (refer to Figure 293), then each entry returned by that Discovery controller may be either a Discovery
Log Page Entry or an Extended Discovery Log Page Entry. Extended Discovery Log Page Entries may
contain zero or more extended attributes. If a Discovery controller returns r ecords that include Extended
Discovery Log Page Entries, then that Discovery controller shall set the Extended (EXTEND) bit to ‘1’ in
the Discovery Log Page Flags (DLPF) field. A Centralized Discovery controller (CDC) shall support
returning Extended Discovery Log Page Entries.
If a Discovery controller supports returning only port local Discovery Log Page Entries or port local Extended
Discovery Log Page Entries (i.e., the Port Local Entries Only Supported (PLEOS) bit is set to ‘1’ in the LID
Specific Parameter field as defined in Figure 292) and that Discovery controller processes a Get Log Page
command for this log page with the Port Local Entries Only (PLEO) bit set to ‘1’ in the LSP field (refer to
Figure 293), then that Discovery controller shall return records for only NVM subsystem ports that are
presented through the same NVM subsystem port that received the Get Log Page command. If a Discovery
controller returns records for only NVM subsystem ports that a re presented through the same NVM
subsystem port that received the Get Log Page command, then that Discovery controller shall set the Port
