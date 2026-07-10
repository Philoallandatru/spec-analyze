---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 538
headings: ["8.1.10 Flexible Data Placement", "8.1.10.1 Flexible Data Placement Overview"]
---

# Source page 538

NVM Express® Base Specification, Revision 2.1

516
CNTLID field of a Registered Controller data structure or Registered Controller Extended data structure).
A controller that supports dispersed namespaces and supports reservations (i.e., a controller that has the
Reservations Support (RESERVS) bit set to ‘1’ in the Optional NVM Command Support (ONCS) field of the
Identify Controller data structure (refer to Figure 312)) shall support the DISNSRS bit being set to ‘1’ in each
command in the following list:
• Reservation Acquire (refer to section 7.5);
• Reservation Register (refer to section 7.6);
• Reservation Release (refer to section 7.7); and
• Reservation Report (refer to section 7.8).
If the HDISNS field is set to 1h in the Host Behavior Support feature and the DISNSRS bit is cleared to ‘0’
in any of the commands in the preceding list when submitted to a dispersed namespace, then the controller
shall abort that command with a status code of Namespace Is Dispersed.
 Flexible Data Placement
 Flexible Data Placement Overview
The Flexible Data Placement (FDP) capability is an optional capability which allows the host to reduce write
amplification by aligning user data usage to physical media. The controller supports log pages which
indicate the status of FDP, statistics about the FDP operation, and information the host uses to detect and
correct usage patterns that increase write amplification.
The scope of the Flexible Data Placement capability is an Endurance Group.
The host enables or disables Flexible Data Placement by issuing a Set Features command specifying the
Flexible Data Placement feature (refer to section 5.1.25.1.20) and the FDP configuration (refer to section
5.1.12.1.28) to be applied to an Endurance Group. The host is required to delete all namespaces associated
with the specified Endurance Group before modifying the value of the Flexible Data Placement feature
(e.g., transitioning from disabled to enabled).
If a Set Features command specifies:
• the Flexible Data Placement feature;
• an Endurance Group in which one or more namespaces exist; and
• a different value from the current value of that feature for that Endurance Group,
then the controller shall abort the command with a status code of Command Sequence Error.
If a Set Features command is successful and changes the Feature value, then:
• all events in the FDP Events log page (refer to section 5.1.12.1.31) are cleared; and
• all fields in the FDP Statistics log page (refer to section 5.1.12.1.30) are cleared to 0h.
Refer to section 3.2.4 for the logical view of the non-volatile storage capacity for the Endurance Group with
Flexible Data Placement enabled.
The Namespace Management command (refer to section 8.1.15) is used to create a namespace within the
Endurance Group. All namespaces in an Endurance Group with Flexible Data Placement enabled shall be
associated with the NVM Command Set (refer to Figure 311) (i.e., the Command Set Identifier (CSI) field
in the Namespace Management command shall be cleared to 00h).  The capacity for the user data stored
by a write command to a namespace is allocated from the Reclaim Unit referenced by the Reclaim Unit
Handle and Reclaim Group specified by that write command. The user data for a namespace is allowed to
be in any stored Reclaim Unit within any Reclaim Group within the Endurance Group (refer to Figure 15).
The Namespace Management command specifies a Placement Handle List (refer to the Namespace
Management command in the appropriate I/O Command Set specification). Each entry in that list specifies
the Reclaim Unit Handle associated with the Placement Handle for that entry. The Placement Handles are
numbered from 0h to the number of entries in the list minus 1. Figure 616 shows an example where
Namespace A associates Reclaim Unit Handle 1 with Placement Handle 0. A specific Reclaim Unit Handle
