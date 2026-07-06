---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 313
headings: []
---

# Source page 313

NVM Express® Base Specification, Revision 2.1

291
• a subset of entries specific to the host or Discovery controller requesting the log page.
Each Host Extended Discovery Log Page Entry shall contain at least one extended attribute containing a
Host Identifier.
Figure 297 specifies the format for the LID Specific Parameter field in the Supported Log Pages log page
(refer to section 5.1.12.1.1) for the Host Discovery log page.
Figure 297: Host Discovery – LID Specific Parameter Field
Bits Description
15:1 Reserved
0
All Host Entries Supported (ALLHOSTES):  If this bit is set to ‘1’, then the Discovery controller supports
returning records for all hosts that are registered without filtering the list of returned records based upon the
requesting HOSTNQN. If this bit is cleared to ‘0’, then the Discovery controller does not support returning
records for all hosts that are registered.
If a Discovery controller supports returning records for all hosts that are registered (i.e., the All Host Entries
Supported (ALLHOSTES) bit is set to ‘1’ in the LID Specific Parameter field as defined in Figure 297) and
that Discovery controller processes a Get Log Page command for this log page with the All Host Entries
(ALLHOSTE) bit set to ‘1’ in the Log Specific Parameter field (refer to Figure 298), then that Discovery
controller shall return records for all hosts that are registered. The list of records returned by the Discovery
controller shall not be filtered based upon the requesting HOSTNQN. If a Discovery controller returns
records for all ho sts that are registered, then that Discovery controller shall set the All Hosts (ALLHOST)
bit to ‘1’ in the Host Discovery Log Page Flags (HDLPF) field. A Discovery controller may require
authentication before reporting the ALLHOSTES bit being set to ‘1’ or may require configuration outside the
scope of this specification.
The Log Page Offset may be used to retrieve specific records. If the Discovery controller supports an index
offset for this log page (i.e., the Index Offset Supported (IOS) bit is set to ‘1’ in the Supported Log Pages
log page (refer to section 5.1.12.1.1) for this log page), then the index values specified in the Log Page
Offset Lower (LPOL) field (refer to Figure 199) and Log Page Offset Upper (LPOU) field (refer to Figure
200) in the Get Log Page command shall be used to index into the header and list of entries within this log
page (e.g., specifying an index offset of 0 returns this log page starting at the header, specifying an index
offset of 1 returns this log page startin g at the offset of Entry 0, etc.). The number of records is returned in
the header of the log page. The format for a Host Extended Discovery Log Page Entry is defined in Figure
299. The format for an extended attribute is defined in Figure 499. The format for the Host Discovery log
page is defined in Figure 300.
A single Get Log Page command used to read the Host Discovery log page shall be atomic. If the Host
Discovery log page is read using multiple Get Log Page commands, then the reader should validate the
Generation Counter field to ensure that there has not b een a change in the contents of the data. The Host
Discovery log page contents should be processed in the following sequence:
1. read the Host Discovery log page contents in order (i.e., with increasing Log Page Offset values);
2. re-read the Generation Counter field after the entire log page is transferred; and
3. if the Generation Counter field does not match the original value read, then discard the log page
read, as the entries may be inconsistent, and restart the sequence; or
4. if the Generation Counter field does match the original value read, then process the returned data.
If the log page contents change during this command sequence, then the Discovery controller may return
a status code of Discover Restart.
Figure 298: Host Discovery Log Specific Parameter Field
Bits Description
14:9 Reserved
08
All Host Entries (ALLHOSTE): If this bit is set to ‘1’, then the Discovery controller shall return records for
all hosts that are registered without filtering the list of returned records based upon the requesting
HOSTNQN. If this bit is cleared to ‘0’, then this bit has no effect.
