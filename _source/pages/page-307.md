---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 307
headings: []
---

# Source page 307

NVM Express® Base Specification, Revision 2.1

285
Local (PORTLCL) bit to ‘1’ in the DLPF field. Only a Direct Discovery controller (DDC) may support returning
only port local Discovery Log Page Entries or port local Extended Discovery Log Page Entries.
If a Discovery controller supports returning records for all NVM subsystem ports that are registered (i.e.,
the All NVM Subsystems Entries Supported (ALLSUBES) bit is set to ‘1’ in the LID Specific Parameter field
as defined in Figure 292) and that Discovery controller processes a Get Log Page command for this log
page with the All NVM Subsystem Entries (ALLSUBE) bit set to ‘1’ in the LSP field (refer to Figure 293),
then that Discovery controller shall return records for all NVM subsystem ports that are registered. The list
of records returned by that Discovery controller shall not be filtered based upon the requesting HOSTNQN.
If a Discovery controller returns records for all NVM subsystem ports that are registered, then that Discovery
controller shall set the All Subsystems (ALLSUB) bit to ‘1’ in the DLPF field. A Discovery controller may
require authentication before reporting the ALLSUBES bit being set to ‘1’ or  may require configuration
outside the scope of this specification.
The Log Page Offset may be used to retrieve specific records. If the Discovery controller supports an index
offset for this log page (i.e., the Index Offset Supported (IOS) bit is set to ‘1’ in the Supported Log Pages
log page (refer to section 5.1.12.1.1) for this log page), then the index values specified in the Log Page
Offset Lower (LPOL) field (refer to Figure 199) and Log Page Offset Upper (LPOU) field (refer to Figure
200) in the Get Log Page command shall be used to index into the header and list of entries within this log
page (e.g., specifying an index offset of 0 returns this log page starting at the header, specifying an index
offset of 1 returns this log page startin g at the offset of Entry 0, etc.). The number of records is returned in
the header of the log page. The format for a Discovery Log Page Entry is defined in Figure 294. The format
for an Extended Discovery Log Page Entry is defined in Figure 295. The format for an extended attribute is
defined in Figure 499. The format for the Discovery log page is defined in Figure 296.
A single Get Log Page command used to read the Discovery log page shall be atomic. If the Discovery log
page is read using multiple Get Log Page commands, then the reader should validate the Generation
Counter field to ensure that there has not been a chan ge in the contents of the data. The Discovery log
page contents should be processed in the following sequence:
1. read the Discovery log page contents in order (i.e., with increasing Log Page Offset values);
2. re-read the Generation Counter field after the entire log page is transferred; and
3. if the Generation Counter field does not match the original value read, then discard the log page
read, as the entries may be inconsistent, and restart the sequence.; or
4. if the Generation Counter field does match the original value read, then process the returned data.
If the log page contents change during this command sequence, then the controller may return a status
code of Discover Restart.
Every record indicates via the SUBTYPE field if that record is describing the current Discovery Service,
referring to another Discovery Service, or if the record indicates an NVM subsystem composed of
controllers that may expose namespaces. A referral to another Discovery Service (i.e., SUBTYPE field set
to 01h) is a mechanism to find additional Discovery subsystems. An NVM subsystem entry (i.e., SUBTYPE
field set to 02h) is a mechanism to find NVM subsystems that contain controllers that may expose
namespaces. An entry that describes the current Discovery Service (i.e., SUBTYPE field set to 03h) is a
mechanism to find additional access information (e.g., other NVM subsystem ports) for the current
Discovery service. Hosts or Discovery controllers are not req uired to process referral entries deeper than
eight levels.
For entries that describe the current Discovery Service, the DUPRETINFO bit set to ‘1’ in the Entry Flags
field indicates those ports that return duplicate information. A host need not retrieve the Discovery Log Page
from all NVM subsystem ports described by a Discovery Log Page Entry that contains the DUPRETINFO
bit set to ‘1’ (refer to Figure 294), as each of those ports return the same information. A host may retrieve
the Discovery Log Page from multiple ports with the DUPRETINFO bit set to ‘1’ to act as redundant access
to the same information. A host may choose to enable the Discovery Log Page Change AEN on one of the
connections associated with an entry with the DUPRETINFO bit set to ‘1’. If the Discovery Log Page Change
AEN is enabled on multiple ports represented by entries with the DUPRETINFO bit set to ‘1’, then the host
may process a Disco very Log Page Change AEN for one connection as applying to all ports represented
by entries that also have the DUPRETINFO bit set to ‘1’.
