---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 543
headings: ["8.1.10.3 Write Commands using Flexible Data Placement (Informative)", "8.1.11 Host-Initiated Refresh Operation"]
---

# Source page 543

NVM Express® Base Specification, Revision 2.1

521
3) Issue Set Features commands for the FDP Events feature (refer to section 5.1.25.1.21) to specify
if supported FDP events are required for Reclaim Unit Handles.
4) For each namespace in the Endurance Group that the Data Placement Directive is to be utilized
by the host by write commands specifying that namespace:
a. Issue an I/O Management Receive command to determine the Placement Handles for that
namespace; and
b. Issue an Identify command with CNS value 08h (i.e., the I/O Command Set Independent
Identify Namespace data structure) to determine if a volatile write cache is present for the
namespace.
 Write Commands using Flexible Data Placement (Informative)
Hosts writing user data to a namespace created in an Endurance Group that has Flexible Data Placement
enabled have two mechanisms for specifying the placement of the user data with the Endurance Group to
the controller.
1. Each namespace has a default Placement Handle. If any write command:
a. has the DTYPE field being ignored by the controller (refer to section 8.1.8.4); or
b. has the DTYPE field not being ignored by the controller and specifies a DTYPE field value
cleared to 00h,
then the Placement Handle value of 0h for the specified namespace is combined with a Reclaim
Group selected by the controller to create the Placement Identifier (refer to  Figure 282 and Figure
283) utilized for that write command.
2. If any write command has the DTYPE field not being ignored by the controller and specifies a
DTYPE field set to Data Placement (i.e., 02h) and specifies a namespace in which the Data
Placement Directive is enabled, then the Placement Identifier specified by the DSPEC field
specifies the Reclaim Group Identifier and Placement Handle used for the write command.
To reduce write amplification, hosts track the user data written to an entire Reclaim Unit and deallocate that
user data by issuing one or more Dataset Management commands specifying the AD bit set to ‘1’ and one
or more Range fields specifying the LBA ranges that were written to that Reclaim Unit.
 Host-Initiated Refresh Operation
The Host-Initiated Refresh operation of the Device Self -test command performs implementation -specific
refresh operations that verify the media integrity and ensure access to media (e.g., media in an NVM
subsystem that has not been in operation for a long p eriod of time and/or has been subject to extreme
environmental conditions). Examples of refresh operations may include read verification, rewrite of
underlying media that are exhibiting a high correctable error rate (e.g., to prevent future errors), and ot her
maintenance activities. A Host-Initiated Refresh operation does not change stored user data.
If the Host -Initiated Refresh operation is supported, then the controller shall set the HIRS bit to ‘1’ in the
DSTO field in the Identify Controller data structure (refer to Figure 312).
The percentage complete of the Host -Initiated Refresh operation is indicated in the Current Percentage
Complete field in the Device Self-test log page (refer to section 5.1.12.1.7).
For Host-Initiated Refresh operation, the NSID field in the Device Self -test command is ignored by the
controller and all media in the NVM subsystem is refreshed regardless of whether any portion of the media
is or is not part of any namespace.
A Host-Initiated Refresh operation shall be aborted:
a) if any Controller Level Reset affects the controller on which the device self-test is being performed;
b) if a Format NVM command is processed by any controller in the NVM subsystem ;
c) if a sanitize operation is started (refer to section 5.1.22); or
d) if a Device Self-test command with the Self-Test Code field set to Fh is processed by any controller
in the NVM subsystem.
