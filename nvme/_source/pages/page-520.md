---
source: "NVM-Express-Base-Specification-Revision-2.1-2024.08.05-Ratified.pdf"
page: 520
headings: ["8.1.7.2 Extended Device Self-Test Operation", "8.1.8 Directives"]
---

# Source page 520

NVM Express® Base Specification, Revision 2.1

498
Figure 596: Format NVM command Aborting a Device Self-Test Operation
SES FNS Bit SENS Bit NSID in Format NVM
command
NSID in Device Self-test
command
Abort Device
Self-Test
operation?
000b
(i.e.,
not a
secure
erase)
0
N/A
Any allocated NSID value
(refer to section 3.2.1.3)
Any active NSID value
(refer to section 3.2.1.4)
Yes, if the NSID
values are the
same
0 FFFFFFFFh Any active NSID value
(refer to section 3.2.1.4) Yes
0 Any allocated NSID value
(refer to section 3.2.1.3) FFFFFFFFh Optional
0 FFFFFFFFh FFFFFFFFh Yes
1 Ignored Ignored Yes
001b
or
010b
(i.e.,
secure
erase)
N/A
0 Any allocated NSID value
(refer to section 3.2.1.3)
Any active NSID value
(refer to section 3.2.1.4
Yes, if the NSID
values are the
same
0 FFFFFFFFh Any active NSID value
(refer to section 3.2.1.4) Yes
0 Any allocated NSID value
(refer to section 3.2.1.3 FFFFFFFFh Optional
0 FFFFFFFFh FFFFFFFFh Yes
1 Ignored Ignored Yes
Key:
Optional = The device self-test operation is not required to be aborted but may be aborted.
 Extended Device Self-Test Operation
An extended device self-test operation should complete in the time indicated in the Extended Device Self -
test Time field in the Identify Controller data structure or less. The percentage complete of the extended
device self-test operation is indicated in t he Current Percentage Complete field in the Device Self -test log
page (refer to section 5.1.12.1.7).
An extended device self -test operation shall persist across any Controller Level Reset and shall resume
after completion of the reset or any restoration of power, if any. The segment where the extended device
self-test operation resumes is vendor specific, but implementations should only have to perform tests again
within the last segment that was being tested prior to the reset.
An extended device self-test operation:
a) shall be aborted by a Format NVM command as described in Figure 596;
b) shall be aborted when a sanitize operation is started (refer to section 5.1.22);
c) shall be aborted if a Device Self-test command with the Self-Test Code field set to Fh is processed;
and
d) may be aborted if the specified namespace is removed from the namespace inventory.
 Directives
Directives is a mechanism to enable host and NVM subsystem or controller information exchange. The
Directive Receive command  (refer to section 5.1.6) is used to transfer data related to a specific Directive
Type from the controller to the host. The Directive Send command (refer to section 5.1.7) is used to transfer
data related to a specific Directive Type from the host to the controller. Other commands may include a
Directive Specific value specific for a given Directive Type (e.g., the Write command in the NVM Command
Set).
Support for Directives is optional and is indicated  by the Directives Supported (DIRS) bit  in the Optional
Admin Command Support (OACS) field in the Identify Controller data structure  (refer to Figure 312).
