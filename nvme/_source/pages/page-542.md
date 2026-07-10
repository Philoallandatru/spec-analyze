---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 542
headings: []
---

# Source page 542

NVM Express® Base Specification, Revision 2.1

520
1) Validate that the Flexible Data Placement capability is supported by issuing an Identify command
to access the Identify Controller data structure and checking that the Flexible Data Placement
Support (FDPS) bit is set to ‘1’.
2) Delete any existing namespaces that exist in the Endurance Group where Flexible Data Placement
is to be enabled.
3) Issue a Get Log Page command specifying the FDP Configurations log page (refer to section
5.1.12.1.28). Parse the FDP Configurations List in the returned log page to determine the desired
FDP configuration.
4) Enable Flexible Data Placement utilizing that desired FDP configuration by issuing a Set Features
command specifying:
a. the Flexible Data Placement feature;
b. the Endurance Group Identifier field set to the ENDGID of the Endurance Group in which
Flexible Data Placement is to be enabled;
c. the FDPE bit set to ‘1’ (i.e., enabling Flexible Data Placement); and
d. the Flexible Data Placement Configuration Index field set to the index of the desired FDP
configuration from the FDP Configurations List in the FDP Configurations log page.
5) Issue Get Features commands for the FDP Events feature (refer to section 5.1.25.1.21) to acquire
the list of supported FDP events and the enabled state of each supported FDP event.
6) Issue Set Features commands for the FDP Events feature (refer to section 5.1.25.1.21) to specify
if FDP events are required for Reclaim Unit Handles.
7) Issue an Identify command specifying the Identify Namespace data structure ( i.e., CNS 00h as
defined in Figure 273) to identify which LBA formats are supported.
8) For each namespace to be created in the Endurance Group, issue a Namespace Management
command specifying:
a. the Select field set to the Create operation (refer to the NVM Express Base Specification);
b. the User Data Format; and
c. the Placement Handle List used to define the Reclaim Unit Handle associated with each
Placement Handle of the namespace (refer to the Namespace Management section of the
applicable I/O Command Set specification).
9) For each namespace created in an Endurance Group in which the Data Placement Directive is to
be utilized by the host by write commands specifying that namespace:
a. Issue a Directive Send command specifying:
i. the Directive Operation field set to Enable Directive (i.e., 01h);
ii. the NSID of that namespace;
iii. the DTYPE field in Command Dword 11 set to Identify (i.e., 00h);
iv. the DTYPE field in Command Dword 12 set to Data Placement (i.e., 02h); and
v. the Enable Directive bit set to ‘1’ (i.e., to enable the Data Placement Directive).
b. Issue an Identify command with CNS value 08h (i.e., the I/O Command Set Independent
Identify Namespace data structure) to determine if a volatile write cache is present for the
namespace.
Following a Controller Level Reset, the host performs the following actions before resuming use of Flexible
Data Placement in an Endurance Group:
1) Determine the FDP configuration:
a. Issue a Get Feature command specifying the Flexible Data Placement feature (i.e., 1Dh)
and the E NDGID for the Endurance Group to determine if Flexible Data Placement is
enabled and the Flexible Data Placement Configuration Index.
b. Issue a Get Log Page command specifying the FDP Configurations log page (i.e., Log
Page Identifier 20h).
2) Issue Get Features commands for the FDP Events feature (refer to section 5.1.25.1.21) to acquire
the list of supported FDP events and the enabled state of each supported FDP event.
